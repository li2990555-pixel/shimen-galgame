# 卷王对决 — 开发实施规格说明书

> 本文档供开发团队直接使用。包含页面规格、API 契约、游戏逻辑、数据模型。
> 版本：v2.0 | 日期：2026-06-18

---

## 目录

1. [页面规格与交互](#1-页面规格与交互)
2. [API 契约](#2-api-契约)
3. [数据模型](#3-数据模型)
4. [游戏逻辑与算法](#4-游戏逻辑与算法)
5. [用户流程状态机](#5-用户流程状态机)
6. [组件库](#6-组件库)

---

## 1. 页面规格与交互

### 1.1 页面总览

| 页面 ID | 页面名 | 路由 | 优先级 | 依赖 |
|---------|--------|------|--------|------|
| P01 | 首页大厅 | / | P0 | — |
| P02 | 远征开始 | /campaign/new | P0 | P01 |
| P03 | 备战 | /campaign/strategy | P0 | P02 |
| P04 | 专注 | /campaign/focus | P0 | P03 |
| P05 | 复盘 | /campaign/review | P0 | P04 |
| P06 | 终局结算 | /campaign/result | P0 | P05 |
| P07 | 个人主页 | /profile | P1 | — |
| P08 | 排行榜 | /leaderboard | P1 | — |
| P09 | 好友管理 | /friends | P1 | — |
| P10 | 战书收件箱 | /challenges | P1 | — |

---

### 1.2 P01 首页大厅

#### 布局规格

```
┌──────────────────────────────────┐
│ [状态栏]                          │
│                                    │
│ 卷王对决  白银III          🔔 ⚙️  │ ← 顶栏（h=48）
│                                    │
│ ┌──────────────────────────────┐  │
│ │ ⚔️ 开始远征                    │  │ ← 主 CTA 按钮
│ │ 选择英雄 · 设定赌注 · 匹配对手  │  │    h=60, 圆角14
│ └──────────────────────────────┘  │
│                                    │
│ 📌 当前战书 (2)                    │  ← section标题
│ ┌──────────────────────────────┐  │
│ │ 好友名 · 25min · 剩余 20h    │  │
│ │ 好友名 · 45min · 剩余 52h    │  │
│ └──────────────────────────────┘  │
│                                    │
│ 在线好友 (3/5)            ＋邀请   │
│ [头像1][头像2][头像3][＋]          │  ← 好友栏
│                                    │
│ 最近远征                           │
│ ┌──────────────────────────────┐  │
│ │ 🏆 vs 影子 · 4轮 · 昨天      │  │
│ │ 📊 vs 好友A · 3轮 · 周三      │  │
│ └──────────────────────────────┘  │
│                                    │
│ [首页][对战][数据][我的]            │  ← 底部导航栏
└──────────────────────────────────┘
```

#### 组件规格

**主 CTA 按钮**
```
状态: default / hover / pressed / disabled
样式:
  bg: linear-gradient(135deg, #FF4757, #e63946)
  color: #FFFFFF
  font-size: 16px, weight: 700
  border-radius: 14px
  box-shadow: 0 4px 15px rgba(255,69,87,0.3)
交互: 点击 → 跳转 P02 / 无好友/任务时 disabled
动画: hover → shadow 放大; press → scale(0.97)
```

**好友头像组件**
```
状态: 在线(绿点) / 离线(灰点) / 对战中(红框)
头像: 44×44, 圆角50%, 渐变色背景
点击: 进入好友对话/发起战书
```

**远征记录列表**
```
数据源: GET /api/campaigns?limit=5
每项:
  - 结果图标 (🏆/📊/❌)
  - vs 对手名
  - 轮次数
  - 相对时间 ("昨天" / "周三")
  - 段位变动 (⏫/⏬)
交互: 点击 → 查看该场结算详情
```

#### 交互细节

```
页面加载:
  1. 并行请求: 用户信息 / 好友列表 / 战书列表 / 最近远征
  2. 骨架屏: 三个卡片占位 + 脉冲动画
  3. 主按钮在数据加载完成后可点击

下拉刷新:
  执行: 重新拉取战书列表 + 最近远征
  视觉: 顶部拉环动画

空状态:
  无好友: 显示 "+ 邀请好友" 全屏引导
  无战书: "没有待应战，去发起一场远征吧"
  无记录: "还没有远征记录"
```

---

### 1.3 P03 备战

#### 布局规格

```
┌──────────────────────────────────┐
│ ← 返回       Round 1 备战  120币  │
│                                    │
│ ○ ● ○ ○ (轮次指示器)              │  ← 4个进度点
│                                    │
│ [事件][商店][装备][任务]            │  ← 标签导航
│                                    │
│ ┌──────────────────────────────┐  │
│ │         (Tab 内容区)           │  │  ← min-h: 240
│ │                                │  │
│ └──────────────────────────────┘  │
│                                    │
│ [🎯 准备就绪，开始专注]              │  ← 底部固定按钮
└──────────────────────────────────┘
```

#### 四个标签页

**TAB 1: 事件**
```
初始状态: 卡片背面朝上，显示 🃏 点击翻开
点击后: 翻牌动画 (0.3s flip)
        显示事件内容 (emoji + 标题 + 效果描述)
状态: face-down(初始) / revealed(已翻开)

事件 UI 规格:
  正面: color bg, emoji 48px, title 15px bold, desc 12px
  卡片: border-radius 12px, padding 20px, min-h 100px
```

**TAB 2: 商店**
```
商品网格: 2×2 grid, gap 8px
每项规格:
  icon: 32px
  name: 12px 500
  price: 12px #FFA502
  状态: available / sold(灰化+标记)

刷新按钮: 右上角 "🔄 刷新 (5币) [x/2]""
  点击: 刷新4件商品, 扣5币
  禁用条件: 余0次 / 余额<5

交互: 点击商品卡 → 弹确认 → 扣币 → 加入背包
```

**TAB 3: 装备**
```
装备栏: 3个槽位, 横向排列
  空: dashed border, 56×56
  已装备: solid gold border, 显示图标
  悬停: 显示 × 移除按钮

背包: 已购物品列表
  每项: icon + name + qty
  点击 → 装备到第一个空槽
  背包为空: "背包空，去商店看看吧"
```

**TAB 4: 任务**
```
任务列表: 4个预设任务
每项: checkbox + 任务名 + 难度标签 + 奖励
  checkbox: 18×18, checked → bg #2ED573
  常规任务: tag-r "常规 +10"
  困难任务: tag-y "困难 +20"

交互: 点击 → toggle 选中/取消
限制: 最多选 4 个，最少 0 个
提示: 选 0 个开始时会弹确认
```

#### 状态变更逻辑

```
页面进入:
  1. 重置事件 (face-down)
  2. 重置商店 (生成4件新商品)
  3. 重置装备栏 (保持背包物品)
  4. 重置任务 (全部未选中)
  5. 刷新次数 ← 2

离开页面(开始专注):
  1. 保存当前装备配置
  2. 保存选中的任务
  3. 记录已翻牌的事件
  4. 保存背包状态
  5. 跳转 P04
```

---

### 1.4 P04 专注页面

#### 布局规格

```
┌──────────────────────────────────┐
│ Round 1 · 第1个番茄钟  ×1.0    │  ← 顶栏 h=36
│                                    │
│                                    │
│             25:00                  │  ← 计时器, 60px
│                                    │
│         ████████░░░░░              │  ← 环形进度 svg
│                                    │
│      专注中 · 保持状态              │  ← 状态文案
│                                    │
│                                    │
│  ┌──────────┐   ┌──────────┐     │
│  │  暂停     │   │  结束     │     │  ← 操作按钮
│  └──────────┘   └──────────┘     │
│                                    │
│  📋 今日任务 ▸                    │  ← 折叠区
│                                    │
│                                    │
│ [首页][⚔️专注中][数据][我的]        │  ← 底部导航
└──────────────────────────────────┘
```

#### 专注状态机

```
状态: IDLE → COUNTDOWN → FOCUSING → PAUSED → COMPLETED/INTERRUPTED

IDLE: 初始, 显示"准备就绪"
COUNTDOWN: 3-2-1 倒计时 (3秒)
FOCUSING: 计时进行中, 显示秒数递减
PAUSED: 暂停, 计时冻结, 显示"已暂停"
COMPLETED: 番茄钟自然完成 → 跳转 P05
INTERRUPTED: 用户主动结束 → 跳转 P05 (带中断标记)

时间颗粒: 25:00 → 00:00, 每秒递减
延长器效果: 总时长 +5min (装备了延长器时)
```

#### 计时器组件规格

```
字体: JetBrains Mono / SF Mono, 60px, 700 weight
颜色: gradient(180deg, #E4E8EC → #484f58)
数字格式: MM:SS, 前补零

环形进度:
  svg 100×100, circle r=42, stroke-width=8
  stroke: linearGradient(#FF4757 → #FFA502)
  stroke-dasharray: 263.9
  stroke-dashoffset: 动态计算 (剩余时间/总时间 × 263.9)
  动画: 1s ease 秒级更新

专注倍率:
  计算: elapsed / 60
    < 10min → ×1.0
    10-20min → ×1.2
    20-30min → ×1.5
    > 30min → ×2.0
  显示: 顶栏右侧, 极小字 11px
```

#### 操作按钮规格

```
暂停按钮:
  正常: bg #21262D, border #30363D, text #E4E8EC
  暂停中: bg #21262D, border #FFA502, text #FFA502 "继续"
  点击: toggle 暂停/继续
  暂停中: 计时冻结, 环形进度停转

结束按钮:
  正常: bg #21262D, border #30363D, text #FF4757
  点击: 弹确认框 "确定结束本轮专注？将记录为中断"
  确认后: 跳转 P05, 传递 isInterrupted=true
```

#### 任务折叠区

```
默认: 折叠状态, 仅显示 "📋 今日任务 ▸"
展开: ▾, 显示该轮选中的任务列表
  每项: checkbox(可交互) + 任务名
  点击: toggle 任务完成状态
  计数: 顶栏显示 "完成数/总数"

视觉规格:
  折叠: padding 8px 10px, border-radius 10px
  展开: 同上, 背景 #161B22
```

---

### 1.5 P05 复盘

#### 布局规格

```
┌──────────────────────────────────┐
│ 📊 Round 1 复盘          +35币  │  ← 顶栏
│                                    │
│ ┌────────── 本轮数据 ──────────┐  │
│  专注: 25min  中断: 0  任务: 2/2 │  │
│  倍率: ×1.5  事件: 安静环境生效  │  │
│ └──────────────────────────────┘  │
│                                    │
│ ┌────────── 对手动态 ──────────┐  │
│  👤 影子·白银II                   │  │
│  Round 1: 22min · 中断 2次       │  │
│  ████████████░░ 累计: 22min     │  │
│ └──────────────────────────────┘  │
│                                    │
│ ┌────────── 总比分 ──────────┐   │
│  你: ████████████████  25       │  │
│  对手: ████████████░░  22      │  │
│ └──────────────────────────────┘  │
│                                    │
│ [⏭️ 备战下一轮]                    │  ← 底部按钮
└──────────────────────────────────┘
```

#### 数据展示

```
本轮聚焦数据:
  专注时长: "XXmin" 绿色(>=25) / 黄色(>=15) / 红色(<15)
  中断次数: "X 次" 绿色(0) / 黄色(1) / 红色(>1)
  任务完成: "X/Y" 全部完成高亮
  实际倍率: "×X.X" 金色高亮(>=1.5)

对手信息(有限):
  只显示累计数据, 不显示单轮
  进度条: (对手累计 / 理想最大) × 100%
  段位标签: 显示对手段位

累计比分:
  进度条: 我 vs 对手, 双色对比
  数字: 各自的累计专注时长
  领先指示: 领先方高亮, 落后方灰色
```

---

### 1.6 P06 终局结算

#### 逐步揭晓时序

```
T+0s: 页面进入 → 背景暗幕 → 显示 🏁 远征结束
T+0.5s: Step1 总时长揭晓 (两个大数字弹入)
T+1.5s: Step2 逐轮对比 (列表逐行出现, 每行间隔 0.2s)
T+3.0s: Step3 关键数据 (中断/任务/倍率)
T+4.0s: Step4 KO 判定 (KO 动画 0.6s)
T+5.0s: 段位变动 + 奖励 (数字滚动效果)
T+6.0s: 操作按钮 (再来一局 / 返回大厅)
```

#### KO 动画规格

```
触发条件: 胜负判定完成
动画元素:
  1. 全屏遮罩: bg 渐黑 (0.2s)
  2. KO 大字: "K.O.!" 或 "WINNER!"
     font-size 48px, font-weight 900
     animation: scale(0.3→1.15→1.0), 0.6s ease-out
     color: win → gold gradient / lose → gray
  3. 粒子效果: 8-12 个粒子从中心扩散 (css animation)
  4. 胜负判定: "你赢了！" / "惜败"
```

#### 数据对比组件的响应式规则

```
逐轮对比行:
  label: "R1" 固定宽度 28
  bar-我: flex 1, 按占比显示
  value-我: 数字, 胜者高亮
  vs: "vs" 文本, 固定宽度 20
  value-对手: 数字, 胜者高亮
  bar-对手: flex 1, 反向

bars 计算:
  每轮: 我占比 = 我时长 / max(我时长, 对手时长)
  对手占比 = 对手时长 / max(我时长, 对手时长)

  当双方都是 0 时, 各占 50%
```

---

## 2. API 契约

### 2.1 全局约定

```
Base URL: /api/v1
Content-Type: application/json
认证: Bearer JWT token
分页: ?page=1&limit=20
排序: ?sort=created_at&order=desc
错误格式: { "error": { "code": "xxx", "message": "xxx" } }
```

### 2.2 用户端 API

#### POST /api/v1/auth/register

注册

```
Request:
{
  "phone": "13800138000",
  "nickname": "卷王小明",
  "avatar_url": "https://...",
  "hero_style": "lightning"  // lightning | shield | flame
}

Response 201:
{
  "user_id": "uuid",
  "token": "jwt_token",
  "coins": 200,
  "elo_rating": 600,
  "tier": "bronze_3"
}
```

#### POST /api/v1/auth/login

登录

```
Request:
{
  "phone": "13800138000",
  "code": "123456"  // 短信验证码
}

Response 200:
{
  "user_id": "uuid",
  "token": "jwt_token",
  "expires_at": "2026-07-18T00:00:00Z"
}
```

#### GET /api/v1/users/me

当前用户信息

```
Response 200:
{
  "user_id": "uuid",
  "nickname": "卷王小明",
  "avatar_url": "https://...",
  "hero_style": "lightning",
  "coins": 120,
  "elo_rating": 820,
  "tier": "silver_2",
  "tier_progress": 0.65,        // 升段进度 0-1
  "total_focus_minutes": 7200,
  "total_campaigns": 47,
  "total_wins": 32,
  "win_rate": 0.68,
  "streak": 3,                   // 当前连胜
  "max_streak": 7,                // 历史最高连胜
  "created_at": "2026-05-01T00:00:00Z"
}
```

#### GET /api/v1/users/me/stats

统计信息

```
Response 200:
{
  "today": {
    "focus_minutes": 45,
    "campaigns_completed": 1,
    "coins_earned": 35
  },
  "this_week": {
    "focus_minutes": 300,
    "campaigns_completed": 7,
    "coins_earned": 240,
    "daily_breakdown": [
      { "date": "2026-06-12", "minutes": 120 },
      { "date": "2026-06-13", "minutes": 45 },
      ...
    ]
  },
  "all_time": {
    "focus_minutes": 7200,
    "campaigns_completed": 47,
    "wins": 32,
    "losses": 15,
    "coins_earned": 4800,
    "coins_spent": 3200
  }
}
```

### 2.3 远征 API

#### POST /api/v1/campaigns

创建/开始新远征

```
Request:
{
  "rounds": 4,                    // 3 | 4 | 5
  "bet_amount": 50,
  "hero_style": "lightning"      // 本次覆盖
}

Response 201:
{
  "campaign_id": "uuid",
  "status": "strategy",          // strategy | focus | review | completed
  "current_round": 1,
  "coins": 120,                   // 剩余专注币
  "expires_at": "2026-06-19T00:00:00Z" // 远征过期时间
}
```

#### POST /api/v1/campaigns/{id}/rounds

提交一轮专注数据

```
Request:
{
  "round_number": 1,
  "focus_seconds": 1500,          // 实际专注秒数
  "breaks": 0,                    // 中断次数
  "tasks_completed": 2,
  "tasks_total": 2,
  "event_id": "quiet_env",        // null 如果没翻牌
  "equipment_ids": ["shield", "extend"], // 装备的物品
  "coins_earned": 35,
  "interrupted": false            // 是否主动中断
}

Response 200:
{
  "round_id": "uuid",
  "multiplier": 1.5,             // 本回合实际倍率
  "cumulative_focus_seconds": 1500,
  "cumulative_breaks": 0,
  "opponent_cumulative": {
    "focus_seconds": 1380,
    "breaks": 2
  },
  "is_leading": true
}
```

#### GET /api/v1/campaigns/{id}

获取远征详情

```
Response 200:
{
  "campaign_id": "uuid",
  "user_id": "uuid",
  "rounds": 4,
  "bet_amount": 50,
  "hero_style": "lightning",
  "status": "completed",         // strategy | focus | review | completed
  "current_round": 4,
  "total_focus_seconds": 6000,
  "total_breaks": 1,
  "total_tasks_completed": 8,
  "total_tasks_total": 9,
  "max_multiplier": 2.0,
  "rounds_data": [
    {
      "round_number": 1,
      "focus_seconds": 1500,
      "breaks": 0,
      "tasks_completed": 2,
      "tasks_total": 2,
      "multiplier": 1.2,
      "coins_earned": 35
    },
    ...
  ],
  "opponent": {
    "id": "uuid",
    "nickname": "影子",
    "tier": "silver_2",
    "total_focus_seconds": 5400,
    "total_breaks": 6,
    "total_tasks_completed": 6,
    "total_tasks_total": 9,
    "max_multiplier": 1.5,
    "rounds_data": [ ... ]
  },
  "result": "win",               // win | lose | draw
  "elo_change": 22,
  "coins_change": 100,
  "created_at": "...",
  "completed_at": "..."
}
```

### 2.4 商店 API

#### GET /api/v1/shop/items

获取所有可用物品

```
Response 200:
{
  "items": [
    {
      "id": "extend",
      "name": "番茄钟延长器",
      "description": "专注时长 +5min",
      "icon": "⏱",
      "price": 30,
      "type": "consumable",      // consumable | permanent
      "category": "time"         // time | defense | attack | economy
    },
    ...
  ]
}
```

#### POST /api/v1/campaigns/{id}/shop/purchase

购买物品

```
Request:
{
  "item_id": "shield"
}

Response 200:
{
  "success": true,
  "coins_remaining": 95,
  "item": { "id": "shield", "name": "专注护盾", "qty": 1 }
}
```

#### POST /api/v1/campaigns/{id}/shop/refresh

刷新商店

```
Request:
{}

Response 200:
{
  "items": [ ... ],              // 新刷新的4件物品
  "coins_remaining": 115,
  "refreshes_left": 1
}
```

### 2.5 好友 API

#### GET /api/v1/friends

好友列表

```
Response 200:
{
  "friends": [
    {
      "user_id": "uuid",
      "nickname": "小明",
      "avatar_url": "...",
      "tier": "silver_3",
      "is_online": true,
      "bond_level": 3,
      "last_active": "2026-06-18T09:00:00Z"
    },
    ...
  ]
}
```

#### POST /api/v1/challenges

下战书

```
Request:
{
  "friend_id": "uuid",
  "campaign_id": "uuid",         // 用哪场远征的数据
  "message": "来战！"
}

Response 201:
{
  "challenge_id": "uuid",
  "status": "pending",           // pending | accepted | declined | expired
  "expires_at": "2026-06-21T00:00:00Z" // 72h 后
}
```

#### GET /api/v1/challenges

战书列表

```
Response 200:
{
  "incoming": [
    {
      "challenge_id": "uuid",
      "from_user": { "nickname": "小明", "tier": "silver_3", "avatar_url": "..." },
      "target_data": { "focus_seconds": 6000, "breaks": 2, "rounds": 4 },
      "message": "来战！",
      "status": "pending",
      "created_at": "...",
      "expires_at": "..."
    }
  ],
  "outgoing": [ ... ]
}
```

### 2.6 天梯匹配 API

#### POST /api/v1/matchmaking/start

开始匹配

```
Request:
{
  "rounds": 4,                   // 想打几轮
  "bet_amount": 50
}

Response 200:
{
  "match_id": "uuid",
  "opponent_ghost_id": "uuid",
  "opponent_tier": "silver_3",
  "status": "matched"
}
```

---

## 3. 数据模型

### 3.1 完整表结构

```sql
-- ── 用户 ──

CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  phone VARCHAR(16) UNIQUE NOT NULL,
  nickname VARCHAR(32) NOT NULL,
  avatar_url TEXT,
  hero_style VARCHAR(16) NOT NULL DEFAULT 'lightning',
  coins INT NOT NULL DEFAULT 200 CHECK (coins >= 0),
  elo_rating INT NOT NULL DEFAULT 600 CHECK (elo_rating >= 0),
  tier VARCHAR(16) NOT NULL DEFAULT 'bronze_3',
  total_focus_minutes INT NOT NULL DEFAULT 0,
  total_campaigns INT NOT NULL DEFAULT 0,
  total_wins INT NOT NULL DEFAULT 0,
  total_losses INT NOT NULL DEFAULT 0,
  streak INT NOT NULL DEFAULT 0,
  max_streak INT NOT NULL DEFAULT 0,
  daily_coins_earned INT NOT NULL DEFAULT 0,  -- 今日已赚币, 每日重置
  daily_coins_date DATE,                        -- 记录日期用于重置
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_users_elo ON users(elo_rating);
CREATE INDEX idx_users_tier ON users(tier);
CREATE INDEX idx_users_phone ON users(phone);


-- ── 远征 ──

CREATE TABLE campaigns (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  opponent_ghost_id UUID,                       -- 匹配的 Ghost ID
  rounds INT NOT NULL CHECK (rounds IN (3,4,5)),
  bet_amount INT NOT NULL DEFAULT 0 CHECK (bet_amount IN (0,10,50,200)),
  hero_style VARCHAR(16) NOT NULL,
  status VARCHAR(16) NOT NULL DEFAULT 'strategy',
  -- strategy → focus → review → completed (逐轮循环)
  current_round INT NOT NULL DEFAULT 1,
  total_focus_seconds INT DEFAULT 0,
  total_breaks INT DEFAULT 0,
  total_tasks_completed INT DEFAULT 0,
  total_tasks_total INT DEFAULT 0,
  max_multiplier DECIMAL(3,1) DEFAULT 1.0,
  result VARCHAR(8),                            -- win / lose / draw / null
  elo_change INT DEFAULT 0,
  coins_change INT DEFAULT 0,
  coins_spent INT DEFAULT 0,                    -- 本场商店消费
  expires_at TIMESTAMPTZ NOT NULL,              -- 72h 内完成
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  completed_at TIMESTAMPTZ
);

CREATE INDEX idx_campaigns_user ON campaigns(user_id);
CREATE INDEX idx_campaigns_status ON campaigns(status);
CREATE INDEX idx_campaigns_completed ON campaigns(completed_at) WHERE status='completed';


-- ── 轮次 ──

CREATE TABLE rounds (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  campaign_id UUID NOT NULL REFERENCES campaigns(id) ON DELETE CASCADE,
  round_number INT NOT NULL CHECK (round_number BETWEEN 1 AND 5),
  focus_seconds INT NOT NULL DEFAULT 0,
  breaks INT NOT NULL DEFAULT 0,
  tasks_completed INT DEFAULT 0,
  tasks_total INT DEFAULT 0,
  multiplier DECIMAL(3,1) DEFAULT 1.0,
  event_id VARCHAR(32),                         -- null if not flipped
  event_effect JSONB,                           -- 事件实际效果记录
  equipment_used JSONB,                         -- ["shield", "extend"]
  coins_earned INT DEFAULT 0,
  was_interrupted BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(campaign_id, round_number)
);


-- ── 物品库存 ──

CREATE TABLE inventory (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  item_id VARCHAR(32) NOT NULL,
  quantity INT NOT NULL DEFAULT 1 CHECK (quantity >= 0),
  acquired_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(user_id, item_id)
);

CREATE INDEX idx_inventory_user ON inventory(user_id);


-- ── 轮次装备快照 ──

CREATE TABLE round_equipment (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  campaign_id UUID NOT NULL REFERENCES campaigns(id) ON DELETE CASCADE,
  round_number INT NOT NULL,
  slot_1 VARCHAR(32),    -- item_id or null
  slot_2 VARCHAR(32),
  slot_3 VARCHAR(32),
  UNIQUE(campaign_id, round_number)
);


-- ── Ghost ──

CREATE TABLE ghosts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  campaign_id UUID NOT NULL REFERENCES campaigns(id),
  rounds INT NOT NULL,
  elo_rating INT NOT NULL,
  tier VARCHAR(16) NOT NULL,
  total_focus_seconds INT NOT NULL,
  total_breaks INT NOT NULL,
  total_tasks_completed INT NOT NULL,
  total_tasks_total INT NOT NULL,
  max_multiplier DECIMAL(3,1) NOT NULL,
  rounds_data JSONB NOT NULL,                    -- 每轮数据详情
  equipment_summary JSONB,                       -- 装备使用统计
  is_active BOOLEAN DEFAULT true,               -- 是否仍可被匹配
  matched_count INT DEFAULT 0,                   -- 已被匹配次数
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  expires_at TIMESTAMPTZ NOT NULL                -- 7 天后过期
);

CREATE INDEX idx_ghosts_elo ON ghosts(elo_rating, rounds) WHERE is_active=true;
CREATE INDEX idx_ghosts_expires ON ghosts(expires_at) WHERE is_active=true;


-- ── 战书 ──

CREATE TABLE challenges (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  from_user_id UUID NOT NULL REFERENCES users(id),
  to_user_id UUID NOT NULL REFERENCES users(id),
  source_campaign_id UUID NOT NULL REFERENCES campaigns(id), -- 作为标的的远征
  message VARCHAR(50),
  status VARCHAR(16) NOT NULL DEFAULT 'pending',
  -- pending | accepted | declined | expired | completed
  result_campaign_id UUID REFERENCES campaigns(id), -- 应战方完成的远征
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  expires_at TIMESTAMPTZ NOT NULL                -- 72h
);

CREATE INDEX idx_challenges_to ON challenges(to_user_id, status);
CREATE INDEX idx_challenges_from ON challenges(from_user_id, status);


-- ── 成就 ──

CREATE TABLE achievements (
  id VARCHAR(32) PRIMARY KEY,                    -- 'first_win', 'perfect_campaign'
  name VARCHAR(64) NOT NULL,
  description TEXT NOT NULL,
  icon VARCHAR(4) NOT NULL,
  category VARCHAR(16) NOT NULL,                 -- battle | streak | social | special
  condition JSONB NOT NULL                       -- 解锁条件的结构化描述
);

CREATE TABLE user_achievements (
  user_id UUID NOT NULL REFERENCES users(id),
  achievement_id VARCHAR(32) NOT NULL REFERENCES achievements(id),
  unlocked_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  PRIMARY KEY (user_id, achievement_id)
);


-- ── 好友关系 ──

CREATE TABLE friendships (
  user_id UUID NOT NULL REFERENCES users(id),
  friend_id UUID NOT NULL REFERENCES users(id),
  bond_level INT NOT NULL DEFAULT 1,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  PRIMARY KEY (user_id, friend_id),
  CHECK (user_id < friend_id)                    -- 避免重复, 保证 user_id < friend_id
);

CREATE INDEX idx_friendships_user ON friendships(user_id);
```

### 3.2 ELO 计算函数

```sql
-- ELO 计算函数
-- 调用方式: SELECT calculate_elo(600, 650, 1, 20);
-- 参数: player_elo, opponent_elo, result(1=win,0=lose,0.5=draw), k_factor

CREATE OR REPLACE FUNCTION calculate_elo(
  player_elo INT,
  opponent_elo INT,
  result DECIMAL(2,1),        -- 1.0 win, 0.0 lose, 0.5 draw
  k_factor INT DEFAULT 20
) RETURNS INT AS $$
DECLARE
  expected DECIMAL(5,3);
  new_elo INT;
BEGIN
  -- 计算预期胜率
  expected := 1.0 / (1.0 + 10.0 ^ ((opponent_elo - player_elo) / 400.0));
  -- 计算新 ELO
  new_elo := player_elo + round(k_factor * (result - expected));
  -- ELO 不低于 0
  RETURN GREATEST(new_elo, 0);
END;
$$ LANGUAGE plpgsql;
```

### 3.3 段位计算函数

```sql
CREATE OR REPLACE FUNCTION calculate_tier(elo INT)
RETURNS TABLE(tier VARCHAR(16), tier_progress DECIMAL(3,2))
LANGUAGE plpgsql AS $$
BEGIN
  RETURN QUERY
  SELECT
    CASE
      WHEN elo >= 2200 THEN 'king_1'
      WHEN elo >= 1900 THEN 'master_' || LEAST(((elo - 1900) / 150) + 1, 2)::TEXT
      WHEN elo >= 1600 THEN 'diamond_' || LEAST(((elo - 1600) / 100) + 1, 3)::TEXT
      WHEN elo >= 1300 THEN 'platinum_' || LEAST(((elo - 1300) / 100) + 1, 3)::TEXT
      WHEN elo >= 1000 THEN 'gold_' || LEAST(((elo - 1000) / 100) + 1, 3)::TEXT
      WHEN elo >= 600 THEN 'silver_' || LEAST(((elo - 600) / 133) + 1, 3)::TEXT
      ELSE 'bronze_' || LEAST((elo / 200) + 1, 3)::TEXT
    END as tier,
    CASE
      WHEN elo >= 2200 THEN 1.0
      WHEN elo >= 1900 THEN ((elo - 1900)::DECIMAL / 150)
      WHEN elo >= 1600 THEN ((elo - 1600)::DECIMAL / 100)
      WHEN elo >= 1300 THEN ((elo - 1300)::DECIMAL / 100)
      WHEN elo >= 1000 THEN ((elo - 1000)::DECIMAL / 100)
      WHEN elo >= 600 THEN ((elo - 600)::DECIMAL / 133)
      ELSE (elo::DECIMAL / 200)
    END as tier_progress;
END;
$$;
```

---

## 4. 游戏逻辑与算法

### 4.1 专注得分计算

```javascript
/**
 * 计算一轮专注的得分
 * 
 * @param {Object} params
 * @param {number} params.focusSeconds  实际专注秒数
 * @param {number} params.targetSeconds 目标专注秒数（基础1500 + 延长器300）
 * @param {number} params.breaks        中断次数
 * @param {number} params.tasksCompleted 完成的任务数
 * @param {number} params.tasksTotal     总任务数
 * @param {string[]} params.equipmentIds 装备的物品 ID 列表
 * @param {number} params.eventMultiplier 事件倍率（默认 1.0）
 * @returns {Object} { coins, multiplier, score }
 */
function calculateRoundScore(params) {
  const {
    focusSeconds,
    targetSeconds,
    breaks,
    tasksCompleted,
    tasksTotal,
    equipmentIds = [],
    eventMultiplier = 1.0
  } = params;

  // 1. 专注倍率（按连续时间递增）
  const elapsedMinutes = focusSeconds / 60;
  let multiplier;
  if (elapsedMinutes < 10) multiplier = 1.0;
  else if (elapsedMinutes < 20) multiplier = 1.2;
  else if (elapsedMinutes < 30) multiplier = 1.5;
  else multiplier = 2.0;

  // 事件倍率叠加
  const effectiveMultiplier = multiplier * eventMultiplier;

  // 2. 基础专注币（按分钟计）
  let coins = Math.floor(elapsedMinutes);

  // 3. 装备效果
  const hasDouble = equipmentIds.includes('double');
  const hasScroll = equipmentIds.includes('scroll');

  // 双倍卡: 专注币 ×2
  if (hasDouble) coins *= 2;

  // 4. 任务奖励
  let taskBonus = 0;
  if (tasksTotal > 0) {
    taskBonus = Math.round((tasksCompleted / tasksTotal) * 20);
    // 任务加速: 任务奖励 +50%
    if (hasScroll) taskBonus = Math.round(taskBonus * 1.5);
  }
  coins += taskBonus;

  // 5. 专注护盾: 有中断时减免损失
  if (breaks > 0 && equipmentIds.includes('shield')) {
    coins += Math.min(breaks * 5, 10); // 护盾补偿
  }

  // 6. 中断惩罚
  if (breaks > 0 && !equipmentIds.includes('shield')) {
    coins = Math.max(0, coins - breaks * 5);
  }

  // 7. 事件效果
  coins = Math.round(coins * eventMultiplier);

  return {
    coins,
    multiplier: effectiveMultiplier,
    score: Math.round(elapsedMinutes * effectiveMultiplier - breaks * 2 + taskBonus)
  };
}
```

### 4.2 远征结算算法

```javascript
/**
 * 结算一场远征
 * 
 * @param {Object} campaign - 当前用户的远征数据
 * @param {Object} ghost - 对手 Ghost 数据
 * @returns {Object} { result, eloChange, coinsChange }
 */
function settleCampaign(campaign, ghost) {
  const myTotal = campaign.total_focus_seconds;
  const oppTotal = ghost.total_focus_seconds;
  
  // 主要胜负判定: 总专注时长
  let result; // 'win' | 'lose' | 'draw'
  if (myTotal > oppTotal + 60) result = 'win';        // 领先超过1min
  else if (oppTotal > myTotal + 60) result = 'lose';
  else result = 'draw';
  
  // 中断作为次要根据
  if (result === 'draw') {
    if (campaign.total_breaks < ghost.total_breaks) result = 'win';
    else if (campaign.total_breaks > ghost.total_breaks) result = 'lose';
    else result = 'draw';
  }

  // ELO 变动
  const kFactor = campaign.total_campaigns <= 10 ? 40 : 20;
  const eloResult = result === 'win' ? 1.0 : result === 'lose' ? 0.0 : 0.5;
  const eloChange = calculateElo(campaign.elo_rating, ghost.elo_rating, eloResult, kFactor);

  // 专注币变动
  let coinsChange = result === 'win' ? campaign.bet_amount * 2
    : result === 'lose' ? -campaign.bet_amount
    : 0; // draw no win/loss

  // 连胜加成 (连胜护符)
  if (result === 'win' && campaign.equipment_used?.includes('amulet')) {
    coinsChange += 20;
  }

  // 状态更新
  const newStreak = result === 'win' ? campaign.streak + 1 : 0;
  const newMaxStreak = Math.max(campaign.max_streak, newStreak);

  return {
    result,
    eloChange,
    coinsChange,
    newElo: campaign.elo_rating + eloChange,
    newTier: calculateTier(campaign.elo_rating + eloChange),
    newStreak,
    newMaxStreak
  };
}
```

### 4.3 专注倍率计算公式

```javascript
/**
 * 专注倍率计算
 * 仅在复盘时计算，战中不显示
 * 
 * 规则: 连续专注时间越长，倍率越高
 * 中断会重置倍率计数器
 * 
 * @param {number} continuousMinutes - 本轮内最长连续专注分钟数
 * @returns {number} 倍率 1.0-3.0
 */
function calculateFocusMultiplier(continuousMinutes) {
  if (continuousMinutes < 10) return 1.0;
  if (continuousMinutes < 20) return 1.2;
  if (continuousMinutes < 30) return 1.5;
  if (continuousMinutes < 45) return 2.0;
  if (continuousMinutes < 60) return 2.5;
  return 3.0;
}
```

### 4.4 Ghost 匹配算法

```javascript
/**
 * 为用户匹配 Ghost
 * 
 * 策略: 找段位最接近的可用 Ghost
 * 优先: 同段位 > 差1段 > 差2段
 * 
 * @param {Object} user - 发起匹配的用户
 * @param {number} rounds - 请求的轮次数
 * @returns {Object|null} 匹配到的 Ghost
 */
async function matchGhost(user, rounds) {
  // 范围: user_elo ± 200
  const minElo = Math.max(0, user.elo_rating - 200);
  const maxElo = user.elo_rating + 200;

  // 找匹配对手: ELO 最近 + 轮次相同
  const ghost = await db.query(`
    SELECT * FROM ghosts
    WHERE is_active = true
      AND rounds = $1
      AND elo_rating BETWEEN $2 AND $3
      AND user_id != $4
    ORDER BY ABS(elo_rating - $5)
    LIMIT 1
    FOR UPDATE SKIP LOCKED
  `, [rounds, minElo, maxElo, user.id, user.elo_rating]);

  if (!ghost) return null;

  // 标记 Ghost 已被匹配
  await db.query(`
    UPDATE ghosts SET matched_count = matched_count + 1
    WHERE id = $1
  `, [ghost.id]);

  // 匹配超过 3 次后失效
  if (ghost.matched_count >= 3) {
    await db.query(`UPDATE ghosts SET is_active = false WHERE id = $1`, [ghost.id]);
  }

  // 返回对手摘要（不含完整数据）
  return {
    id: ghost.id,
    tier: ghost.tier,
    elo_rating: ghost.elo_rating,
    total_focus_seconds: ghost.total_focus_seconds,
    total_breaks: ghost.total_breaks,
    // 不返回完整轮次数据（用于信息不对称）
  };
}
```

### 4.5 商店刷新逻辑

```javascript
/**
 * 商店物品刷新
 * 
 * 规则: 从总物品池中随机抽取 4 件
 * 不重复: 同一次刷新中不会出现重复物品
 * 可用率: 保证至少 1 件是用户余额可购买的
 */
function generateShopItems(userCoins, allItems, excludeItems = []) {
  let available = allItems.filter(i => !excludeItems.includes(i.id));
  
  // 洗牌
  for (let i = available.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [available[i], available[j]] = [available[j], available[i]];
  }

  // 取前 4 件
  let items = available.slice(0, 4);

  // 保证至少 1 件可买
  const hasAffordable = items.some(i => i.price <= userCoins);
  if (!hasAffordable && items.length > 0) {
    // 替换最贵的一件为可买的
    const affordable = allItems.filter(i => i.price <= userCoins && !excludeItems.includes(i.id));
    if (affordable.length > 0) {
      const replacement = affordable[Math.floor(Math.random() * affordable.length)];
      items[items.length - 1] = replacement;
    }
  }

  return items;
}
```

---

## 5. 用户流程状态机

### 5.1 远征状态机

```
                  ┌─────────┐
                  │ 初始状态  │
                  └────┬────┘
                       │ 点击"开始远征"
                       ↓
                  ┌─────────┐
           ┌─────→│  备战    │ ←──────┐
           │      │(Strategy)│        │
           │      └────┬────┘        │
           │           │ 点击"开始专注"│
           │           ↓              │
           │      ┌─────────┐        │
           │      │  专注    │        │
           │      │ (Focus)  │        │
           │      └────┬────┘        │
           │           │ 计时结束/中断 │
           │           ↓              │
           │      ┌─────────┐        │
           │      │  复盘    │        │
           │      │ (Review) │        │
           │      └────┬────┘        │
           │           │             │
           │    ┌──────┴──────┐     │
           │    │             │     │
           │  还有下一轮      │     │
           │    │            最后一轮 │
           │    ↓             │     │
           │   ┌───┐          ↓     │
           └───┤ ← ┘    ┌─────────┐ │
                       │ 终局结算  │ │
                       │(Result)  │ │
                       └────┬────┘ │
                            │      │
                            ↓      │
                       ┌─────────┐ │
                       │  完成    │ │
                       └─────────┘ │
                                   │
   异常路径:                        │
   用户中途退出APP → 远征状态保持     │
   恢复时回到上次离开的阶段 ──────────┘
   72h 未完成 → 远征过期, 标记失败
```

### 5.2 战书状态机

```
                  ┌─────────┐
                  │ 已发送   │
                  │(Pending) │
                  └────┬────┘
                       │
            ┌──────────┼──────────┐
            │          │          │
            ↓          ↓          ↓
       ┌──────┐  ┌────────┐  ┌────────┐
       │ 已接受 │  │ 已拒绝  │  │ 已过期  │
       │Accept │  │Decline │  │Expired │
       └──┬───┘  └────────┘  └────────┘
          │
          ↓
     ┌─────────┐
     │对方完成远征│
     │ 对比结算  │
     └────┬────┘
          │
          ↓
     ┌─────────┐
     │  已完成   │
     │Completed │
     └─────────┘
```

### 5.3 日常活跃循环

```
                         ┌──────────┐
                         │ 打开 APP  │
                         └────┬─────┘
                              ↓
                    ┌─────────────────┐
                    │ 首页大厅          │
                    │ 展示: 战书/好友/  │
                    │       最近远征    │
                    └────┬────────────┘
                         │
              ┌──────────┼──────────┐
              │          │          │
              ↓          ↓          ↓
        ┌─────────┐ ┌────────┐ ┌────────┐
        │ 发起远征  │ │ 响应战书│ │影子挑战│
        └────┬────┘ └───┬────┘ └───┬────┘
             │          │          │
             └──────────┼──────────┘
                        ↓
                   ┌─────────┐
                   │  远征流程  │
                   │(4~5轮)   │
                   └────┬────┘
                        ↓
                   ┌─────────┐
                   │ 结算+奖励 │
                   └────┬────┘
                        │
              ┌─────────┼──────────┐
              │         │          │
              ↓         ↓          ↓
        ┌────────┐ ┌───────┐ ┌────────┐
        │ 再来一局│ │下战书  │ │ 退出APP │
        └────────┘ └───────┘ └────────┘
```

---

## 6. 组件库

### 6.1 组件清单

| 组件 | 用途 | 复用页面 |
|------|------|---------|
| Timer | 大字计时器 | P04 |
| CircularProgress | 环形进度条 | P04 |
| ProgressBar | 线性进度条 | P05, P06 |
| CtaButton | 主操作按钮 | 所有页面 |
| Avatar | 用户头像 | P01, P05 |
| FriendRow | 好友横向列表 | P01 |
| TabBar | 标签栏切换 | P03 |
| EventCard | 事件牌 | P03 |
| ShopGrid | 商店网格 | P03 |
| EquipmentSlots | 装备栏 | P03 |
| Inventory | 背包列表 | P03 |
| TaskList | 任务清单 | P03, P04 |
| RoundDots | 轮次指示器 | P03 |
| StatRow | 数据对比行 | P05, P06 |
| Settlement | 结算逐步揭晓 | P06 |
| KOEfect | KO 动画 | P06 |

### 6.2 核心组件 Props

```typescript
// Timer
interface TimerProps {
  seconds: number;           // 剩余秒数
  totalSeconds: number;      // 总秒数
  status: 'countdown' | 'focusing' | 'paused';
  onPause: () => void;
  onResume: () => void;
  onEnd: () => void;
}

// CircularProgress
interface CircularProgressProps {
  progress: number;          // 0-1
  size: number;              // px
  strokeWidth: number;       // px
  color: string;
  backgroundColor: string;
}

// EventCard
interface EventCardProps {
  state: 'face-down' | 'revealed';
  event?: {
    emoji: string;
    title: string;
    description: string;
    type: 'positive' | 'negative' | 'choice';
  };
  onFlip: () => void;
}

// ShopGrid
interface ShopItem {
  id: string;
  icon: string;
  name: string;
  description: string;
  price: number;
}

interface ShopGridProps {
  items: ShopItem[];
  coins: number;
  onPurchase: (itemId: string) => void;
  onRefresh: () => void;
  refreshesLeft: number;
}

// EquipmentSlots
interface EquipmentSlotsProps {
  slots: (string | null)[];  // 3 slots, item_id or null
  maxSlots: number;
  onUnequip: (slotIndex: number) => void;
}

// Settlement
interface SettlementProps {
  steps: SettlementStep[];
  onComplete: () => void;
}

interface SettlementStep {
  delay: number;             // ms from start
  component: React.ReactNode;
  animation: string;         // CSS class name
}

// KOEffect
interface KOEffectProps {
  isWin: boolean;
  onAnimationEnd: () => void;
}
```

### 6.3 动效规范

```
页面切换: fadeIn 0.35s ease, translateY(10px→0)
按钮交互: active → scale(0.96), 0.15s
好友头像: 在线绿点 pulse 2s infinite

翻牌动画:
  0ms: 当前面旋转压缩 (scaleX 1→0), 0.15s
  150ms: 背面旋转展开 (scaleX 0→1), 0.15s

KO 动画:
  0ms: 全屏遮罩 fadeIn, 0.2s
  200ms: KO 大字 scale(0.3→1.1), 0.4s
  600ms: KO 大字 settle scale(1.1→1.0), 0.2s

逐步揭晓:
  每步: fadeIn + translateY(20→0), 0.4s
  间距: 500ms
```

### 6.4 响应式规则

```
基础断点:
  Mobile: 320-768px (MVP 优先)
  Tablet: 769-1024px
  Desktop: 1025px+

手机壳布局:
  最大宽度: 390px
  居中: margin auto
  左右 padding: 16px

Tab 内容区:
  最小高度: 240px
  溢出: scroll (仅在内容超长时)
```

---

## 附录：关键决策记录

| 决策 | 选项 | 最终选择 | 理由 |
|------|------|---------|------|
| 竞技模式 | 同步 / 异步 | **异步 Ghost** | 专注零干扰 + 时间灵活 |
| 一局时长 | 10/25/45min | **25min** | 番茄钟标准 + 足够深度学习 |
| 远征轮次 | 1-10 轮 | **3/4/5** | 3入门,4标准,5深度 |
| 装备机制 | 可购 / 随机掉落 | **可购** | 策略确定性, 非赌博 |
| 赌注意义 | 专注币 / 段位分 | **专注币** | 不影响公平性 |
| 匹配范围 | ±100/200/400 ELO | **±200** | 平衡匹配速度与公平性 |
| Ghost 有效期 | 3/7/30 天 | **7 天** | 数据新鲜度 |
| Ghost 匹配上限 | 1/3/无限次 | **3 次** | 避免过时数据反复使用 |
