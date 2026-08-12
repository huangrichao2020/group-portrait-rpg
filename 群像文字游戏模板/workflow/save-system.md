# 存档系统完整规范 · Save System v1

> 任意 Agent 加载群像文字游戏 skill，**必须**按本规范执行存档与选档。
> 本文档是模板层规范，具体 skill 在 SKILL.md 顶部有快速参考卡，本文件是详细操作手册。

## 0 · 核心理念

**进度永不掉，存档可挑拣**。玩家每一次选择都立刻落盘，下次开打能从断点无缝续玩；同时玩家可主动"另存到 slot N"做多线存档。

## 1 · 工作空间（Work Space）

每个具体 skill 在加载时，DM 必须在以下位置创建/扫描存档目录：

```
{skill 所在目录}/saves/
```

例如：
- `桌面/群像文字游戏/01-饭局-最后一顿散伙饭/saves/save-2026-08-11-1620-001.json`
- `桌面/群像文字游戏/05-同学会-十年再聚/saves/save-2026-08-12-0915-003.json`

**设计理由**：skill 自包含、可随整个文件夹分享、玩家在桌面一眼能看到存档。

## 2 · 存档槽位（Slots）

每个 skill 最多 9 个存档位（slot 1–9）：

| Slot | 用途 | 默认行为 |
|---|---|---|
| **slot 1** | 主自动存档 | 每个分支点完成时自动覆盖 |
| **slot 2** | 死光快照 | 死光节点结束时强制存档到 slot 2 |
| **slot 3–9** | 玩家手动分支存档 | 玩家说"另存到 slot N" |

**slot 满了怎么办**：DM 提示玩家"存档已满，请选一个 slot 覆盖（输入 slot 号覆盖，或输入 'cancel' 取消本次另存）"。

## 3 · 文件命名

```
save-{YYYYMMDD}-{HHMM}-{slot}.json
```

示例：
- `save-20260811-1620-001.json`（2026-08-11 16:20，slot 1）
- `save-20260811-1630-002.json`（死光快照）

**自动备份**：每次覆盖存档前，把上一版本复制为 `save-...-{slot}.bak.json`，便于损坏时回退。

## 4 · 存档数据格式

完整 schema 见 `../fixed/schema.md` 的 `save_data v1` 章节。核心要点：

- **不存原始 game-data**——每次从 SKILL.md/game-data.json 重读
- **必存**：进度（act/branch/completed）+ 世界状态（关系强度/暗线揭示）+ 玩家历史（每步选择/答复/效果）+ 元信息
- **存档类型枚举**：`branch_completed / death_point / pause / ending / auto`

## 5 · 存档时机（强制）

DM **必须**在以下时机更新存档：

| 时机 | 操作 | 提示语 |
|---|---|---|
| **玩家做选择后** | 更新 slot 1 | （静默存，不打扰沉浸） |
| **分支点完成** | 更新 slot 1 + 自动备份 | "✅ 已自动存档（slot 1）" |
| **死光节点结束** | 强制存到 slot 2（专用死光槽位） | "💀 死光快照已存档（slot 2），下次可直接从死光后继续" |
| **玩家说"暂停"** | 标记 `interrupted=true` 存 slot 1 | "⏸️ 暂停存档已保存，下次加载会优先提示恢复" |
| **玩家说"另存到 slot N"** | 强制覆盖 slot N | "💾 已另存到 slot N" |
| **达成结局** | 存档 + `ending_unlocked` 追加新结局 | "🏆 结局已解锁：E1 体面散场" |
| **对话意外中断**（玩家关闭） | 当前存档标记 `interrupted=true` | （下次加载时 DM 主动询问是否继续） |

## 6 · 选档协议（必读）

DM 加载 SKILL.md **第一件事**，必须执行：

### 6.1 扫描存档

```bash
ls -la ./saves/   # 或等效工具
```

列出所有 `save-*.json` 文件，按 `last_played_at` 倒序。

### 6.2 展示存档

如果发现存档，DM 用以下开场白：

> 🗄️ **欢迎回到「{游戏名}」**。
>
> 我在这里发现了 N 个存档：
>
> 1. **[存档 1]** 创建于 2026-08-11 14:30，第 2 幕 B2 节点，已玩 12 分钟，已揭示 1 条暗线
> 2. **[存档 2]** 创建于 2026-08-11 16:20，第 4 幕·死光后，已玩 28 分钟，已揭示 5 条暗线
> 3. ...
>
> 输入 `新` 开始新游戏（用 slot 1 覆盖原存档）。
> 输入 `1-N` 继续对应存档。
> 输入 `删 N` 删除存档 N（需确认）。

### 6.3 加载存档

玩家输入 `1-N` → DM 读对应 `save-{...}-{slot}.json` → 从 `current_act/current_branch` 继续：
- 不重读 `completed_branches` 里的节点
- 重置 `last_played_at`、递增 `play_count`
- 跳过已揭示暗线
- 关系强度表按存档值恢复

### 6.4 没有存档

如果没有存档文件，DM 直接进入新游戏开场白（第 0 幕·布场）。

## 7 · 存档数据写入示例

每次更新存档，DM 用 Python 写入（避免中文 heredoc 问题）：

```python
import json, os
from datetime import datetime

SAVE_DIR = "{skill 所在目录}/saves"
SLOT = 1  # 或 2（死光）, 或玩家指定

save_path = os.path.join(SAVE_DIR, f"save-{datetime.now().strftime('%Y%m%d-%H%M')}-{SLOT:03d}.json")
# 备份上一版本
if os.path.exists(save_path):
    bak_path = save_path.replace('.json', '.bak.json')
    os.rename(save_path, bak_path)

save_data = {
    "$schema": "save_data_v1",
    "save_id": f"save-{datetime.now().strftime('%Y%m%d-%H%M')}-{SLOT:03d}",
    "slot": SLOT,
    "game_id": "{game_id}",
    "created_at": "{原 created_at}",
    "last_played_at": datetime.now().isoformat(timespec='seconds'),
    "play_count": {递增},
    "progress": {
        "current_act": {当前幕},
        "current_branch": "{当前分支}",
        "completed_branches": [...],
        "remaining_branches": [...],
        "trigger_reached": bool,
        "ending_id": null
    },
    "world_state": {
        "revealed_secrets": [...],
        "relationship_table": {...},
        "npc_attitudes": {...}
    },
    "player_history": [...],
    "meta": {
        "play_time_minutes": {累加},
        "ending_unlocked": [...],
        "last_save_type": "{branch_completed/death_point/pause/ending/auto}",
        "interrupted": bool
    }
}

os.makedirs(SAVE_DIR, exist_ok=True)
with open(save_path, "w", encoding="utf-8") as f:
    json.dump(save_data, f, ensure_ascii=False, indent=2)
```

## 8 · 失败处理

DM 必须处理的异常：

| 情况 | DM 行为 |
|---|---|
| 存档 JSON 损坏 | 尝试读 `.bak.json`，如果也损坏则提示玩家该存档不可恢复 |
| 存档 schema 版本不匹配 | 提示玩家存档来自旧版本，建议开始新游戏 |
| `saves/` 目录权限不足 | 提示玩家检查目录权限 |
| 玩家选了不存在的 slot | 重新展示存档列表 |

## 9 · 隐私与迁移

- 存档只在本地 skill 目录，**不上传任何云端**
- 玩家可手动 cp `saves/` 到其他位置备份
- 发给别人时，**默认排除 saves/**（或单独说明"想继续你的进度就把 saves/ 也带上"）

## 10 · 与 DM 操作手册的关系

存档机制是 DM 操作手册的**第 9 节**，与开场/提问/NPC 互动/死光演出/结局判定并列。完整操作见 `dm-operations.md`。