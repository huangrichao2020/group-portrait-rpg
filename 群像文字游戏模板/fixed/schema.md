# 群像文字游戏 · 数据 Schema

> 一份 `game-data.json` 就是一局游戏的"剧本数据库"。本文件定义每个字段的类型、含义与示例。
> 抽取自 novel-evolution 的动态库范式（`relationships.md` 人物关系库 / `world-state.md` 世界状态 / `active-threads.md` 剧情线），并改造为"可玩"形态。

---

## 顶层结构

```jsonc
{
  "meta":   { ... },   // 元信息
  "venue":  { ... },   // 场域 / 裹挟事件 / 世界暗线
  "cast":   [ ... ],   // 角色卡（3–7 人）
  "trigger":{ ... },   // 死光节点
  "branches":[ ... ],  // 分支点 / 玩家代理点
  "endings":[ ... ]    // 结局矩阵
}
```

---

## 1. meta（元信息）

| 字段 | 类型 | 含义 | 示例 |
|------|------|------|------|
| `title` | string | 游戏标题 | "最后一顿散伙饭" |
| `genre` | string | 场景类型，对应 `genres.json` 的 key | "饭局" |
| `duration` | string | 预估时长 | "20–40 分钟 / 一局" |
| `player_role` | string | 玩家"第七人"的身份 | "A 的大学室友，刚回杭州" |
| `tone` | string | 文风基调 | "市井、微醺、暗流" |
| `platforms` | string[] | 适配平台 | ["公众号","小红书","B站"] |

---

## 2. venue（场域 / 裹挟事件 / 世界暗线）

| 字段 | 类型 | 含义 | 示例 |
|------|------|------|------|
| `time` | string | 时间 | "周五晚 7 点，入秋" |
| `place` | string | 地点 | "老城区巷子里的私房菜馆'三点半'" |
| `event` | string | **裹挟事件**——把所有人强塞进同一时空的那件事 | "A 张罗的'随便聚聚'，其实是他离杭前的散伙饭，但除他外无人知情" |
| `world_hidden` | string[] | **世界暗线**——真实发生、玩家未必知道的事 | ["A 已签好外地 offer","B 今晚本要向 C 表白","E 背着 30 万债"] |

> 设计要点：event 要"同一目标"（都来吃饭），但每人"不同目的"（聚友/表白/帮腔/躲债…）。这正是群戏四要素的前两格。

---

## 3. cast（角色卡，3–7 人）

每人一个对象：

| 字段 | 类型 | 含义 | 示例 |
|------|------|------|------|
| `name` | string | 名字 | "A" |
| `role` | string | 局中表面身份 | "组局者，看似主角" |
| `want` | string | **核心诉求**（群戏第一铁律） | "体面地告别，不让人看出自己是逃" |
| `mask` | string | 表面人设 / 伪装 | "豪爽大哥，永远在张罗" |
| `secret` | string | 秘密 / 伏笔 | "其实是被裁后撑不住才走" |
| `tie_to_player` | string | 与"第七人"的恩怨/羁绊 | "你是他四年室友，他知道你最懂他" |
| `arc` | string | 弧光（表层住相→深层住相） | "表面：我撑得住 → 深层：我早就累了" |

> 关系网张力来源：`want` 之间必须**互相啃咬**（B 要表白 ↔ C 来帮 B 却撞上 D 的雷）。这是"互动产生新冲突"的燃料。

---

## 4. trigger（死光节点）

| 字段 | 类型 | 含义 | 示例 |
|------|------|------|------|
| `moment` | string | **所有诉求同时撞车**的那一句话/那一刻 | "A 突然举杯：'这顿吃完，我就离开杭州了。'" |
| `why_deadly` | string | 为何此刻不可逆 | "B 的表白、C 的助攻、E 的借钱，全在同一秒失去意义" |

> "死光"不是真死人，是所有人的小算盘在同一刻被碾碎。这是群戏四要素的最后一格——**然后死光**。

---

## 5. branches（分支点 / 玩家代理点，4–8 个）

每个节点：

| 字段 | 类型 | 含义 | 示例 |
|------|------|------|------|
| `id` | string | 编号 | "B1" |
| `act` | number | 发生在第几幕（1–4） | 2 |
| `scene` | string | 抛给玩家的情境描写 | "B 借上厕所把你拽出来，手在抖：'帮我个忙，待会儿我要当众…'" |
| `prompt` | string | 你问玩家的话 | "你要不要帮 B 这个忙？" |
| `player_must` | string | 玩家被要求给出的"判断/真实答复"类型 | "真实答复：你跟 B 关系如何，愿不愿蹚这浑水" |
| `effects` | object | 不同答复如何改写世界 | `{"帮": "...", "拒": "...", "装傻": "..."}` |

`effects` 的键是玩家可选动作，值是该动作导致的**关系/线索/走向变化**（用自然语言描述，主持人据此推演）。

---

## 6. endings（结局矩阵，3–5 种）

| 字段 | 类型 | 含义 | 示例 |
|------|------|------|------|
| `id` | string | 编号 | "E1" |
| `name` | string | 结局名 | "体面散场" |
| `condition` | string | 触发条件（基于关系强度/线索/玩家立场） | "玩家帮 B 且未拆穿 A，A 平稳离场" |
| `text` | string | 结局正文（沉浸描写） | "门在 A 身后合上…" |

> 结局不是非黑即白，是**关系网收束后的自然落点**。好结局也可能带着刺，坏结局也可能留一线。

---

## 7. save_data（存档数据，运行时生成）

> 存档**不存原始 game-data**——每次从 SKILL.md/game-data.json 重读，存档只存"运行状态"。
> schema 版本号：`save_data_v1`

### 顶层字段

| 字段 | 类型 | 含义 | 示例 |
|------|------|------|------|
| `$schema` | string | 存档 schema 版本 | "save_data_v1" |
| `save_id` | string | 存档唯一 ID | "save-2026-08-11-1620-001" |
| `slot` | number | 存档槽位（1-9） | 1 |
| `game_id` | string | 对应游戏 skill 标识 | "01-饭局-最后一顿散伙饭" |
| `created_at` | string | 创建时间（ISO 8601） | "2026-08-11T16:20:00+08:00" |
| `last_played_at` | string | 最后游玩时间（ISO 8601） | "2026-08-11T16:25:00+08:00" |
| `play_count` | number | 本存档游玩次数 | 1 |

### `progress`（进度）

| 字段 | 类型 | 含义 | 示例 |
|------|------|------|------|
| `current_act` | number | 当前所在幕（1-4） | 2 |
| `current_branch` | string\|null | 当前所在分支点 ID | "B2" |
| `completed_branches` | string[] | 已完成的分支 ID | ["B1"] |
| `remaining_branches` | string[] | 剩余分支 ID | ["B2","B3","B4","B5"] |
| `trigger_reached` | boolean | 是否已抵达死光节点 | false |
| `ending_id` | string\|null | 已达成结局（null=未达） | null |

### `world_state`（世界状态）

| 字段 | 类型 | 含义 | 示例 |
|------|------|------|------|
| `revealed_secrets` | number[] | 已揭示暗线索引（对应 venue.world_hidden[]） | [0] |
| `relationship_table` | object | 关系强度（-100~100），双向 | `{"A-player": 0, "B-player": 15, "A-B": 35}` |
| `npc_attitudes` | object | 每个 NPC 当前情绪关键词 | `{"A": "努力张罗", "B": "紧张想表白"}` |

### `player_history`（玩家历史）

每条记录一个已完成分支：

```jsonc
{
  "branch_id": "B1",
  "scene_rendered": "B 拽住你袖子...",
  "prompt": "你要不要帮 B 这个忙？",
  "player_response": "行，我帮你",
  "choice": "帮",
  "effects_applied": {
    "B-player": "+15",
    "trigger_advance": false
  },
  "dm_reply": "B 松了口气...",
  "timestamp": "2026-08-11T16:21:00+08:00"
}
```

### `meta`（元信息）

| 字段 | 类型 | 含义 | 示例 |
|------|------|------|------|
| `play_time_minutes` | number | 累计游玩分钟 | 12 |
| `ending_unlocked` | string[] | 已解锁结局 ID | [] |
| `last_save_type` | string | 最近存档类型 | "branch_completed" |
| `interrupted` | boolean | 是否中断（玩家中途离开） | false |

### 存档类型枚举 `last_save_type`

| 值 | 触发时机 |
|---|---|
| `branch_completed` | 普通分支点完成 |
| `death_point` | 死光节点快照 |
| `pause` | 玩家主动暂停 |
| `ending` | 达成结局 |
| `auto` | 定时/自动备份 |
