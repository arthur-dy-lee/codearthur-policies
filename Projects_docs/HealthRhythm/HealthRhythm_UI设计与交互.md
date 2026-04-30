# HealthRhythm — UI 设计与交互文档

**版本：** v1.0
**日期：** 2026 年 4 月 16 日
**产品名：** HealthRhythm — Fasting · Calories · Weight
**目标市场：** 海外（英文主 + 简体中文次），首发美区
**技术栈：** iOS 17+ / Swift 5.10 / SwiftUI / SwiftData
**文档性质：** UI wireframe 规范（以 ASCII 线框 + 交互说明为主，视觉稿后续出 Figma）

---

## 目录

1. [设计理念与原则](#1-设计理念与原则)
2. [设计系统](#2-设计系统)
3. [全局导航与框架](#3-全局导航与框架)
4. [首次使用流程 (Onboarding)](#4-首次使用流程-onboarding)
5. [Dashboard 首页 (HomeView)](#5-dashboard-首页-homeview)
6. [断食模块 (FastingView)](#6-断食模块-fastingview)
7. [饮食记录模块 (MealView)](#7-饮食记录模块-mealview)
8. [体重记录模块 (WeightView)](#8-体重记录模块-weightview)
9. [运动记录模块 (WorkoutView)](#9-运动记录模块-workoutview)
10. [减肥计划模块 (PlanView)](#10-减肥计划模块-planview)
11. [设置模块 (SettingsView)](#11-设置模块-settingsview)
12. [AI BYOK 配置页](#12-ai-byok-配置页)
13. [付费墙 (Paywall)](#13-付费墙-paywall)
14. [合规弹窗与警告](#14-合规弹窗与警告)（含 ATT Pre-prompt、断食超长警告、弹窗设计总原则）
15. [Widget 与 Live Activity](#15-widget-与-live-activity)
16. [动画与微交互](#16-动画与微交互)
17. [暗色模式适配](#17-暗色模式适配)
18. [iPad 适配](#18-ipad-适配)
19. [无障碍设计](#19-无障碍设计)
20. [空态/加载/错误](#20-空态加载错误)

---

## 1. 设计理念与原则

### 1.1 设计哲学

**"Lean, not Mean"** — 减肥不该是自我惩罚。界面要传达"轻盈、掌控、进步"的正向感受，而不是"挨饿、焦虑、对抗"。打开 App 3 秒内看到今日全景；操作路径短，0–2 层就能完成记录。

### 1.2 五大设计原则

| 原则 | 说明 | 具体体现 |
|------|------|----------|
| Glanceable（一瞥可懂） | 首页第一屏看清今日状态 | 热量杯 + 断食环 + 体重趋势三件套，无需滚动 |
| Frictionless（无摩擦） | 单次记录 ≤ 3 秒 | 常用收藏、最近记录、AI 一句话录入、一键复制昨日 |
| Progress-forward（进步优先） | 永远强调"接近目标了多少"，不强调"你又胖了多少" | 涨了用"保持观察"、跌了用"棒极了" |
| Safety-first（安全优先） | 合规红线硬编码进 UI 层 | 热量下限警告、BMI 拦截、免责弹窗 |
| Honest（诚实） | 不虚假营销，不用 before/after | 数据曲线为主，文案中性 |

### 1.3 情绪设计 (Emotional Design)

| 阶段 | 情感目标 | 视觉表达 |
|------|---------|---------|
| 打开 App | 安全、清晰 | 一屏全景，无红色警告 |
| 记录中 | 快、顺、无压力 | 大按钮、常用置顶、智能填充 |
| 看到缺口正数（减脂中） | 满足、肯定 | 绿色进度、"Great pace today" 文案 |
| 看到缺口负数（超额） | 不焦虑、不羞耻 | 中性灰色、"Tomorrow is a new day" 文案，**绝不**用红色警告色羞辱 |
| 达成目标 | 庆祝、被看见 | 粒子动画、分享卡片、"You did it" |
| 合规警告 | 严肃、被关怀 | 琥珀色横幅（不是红色恐吓）、"We care about you" 语气 |

### 1.4 绝对不做的视觉选择

- ❌ 前后对比图（before/after）
- ❌ 裸露或近裸的身材照
- ❌ "Lose 10 lbs in 7 days" 之类承诺式营销
- ❌ 红色羞辱（吃超了显红字）
- ❌ 推送"你今天又没记录！"类负面强化
- ❌ 称重数字对比秀（"从 80kg 到 60kg"）

---

## 2. 设计系统

### 2.1 色彩系统（清新健康向，非"节食焦虑"向）

#### 主色板

| Token | 浅色模式 | 暗色模式 | 用途 |
|-------|---------|---------|------|
| `primary` | `#4ECDC4`（薄荷青） | `#5FDDD4` | 品牌色、主 CTA、断食环 |
| `accent` | `#FF8A65`（珊瑚橘） | `#FFA488` | 强调色、热量杯填充、运动 |
| `success` | `#81C784`（草绿） | `#A5D6A7` | 达成、缺口正数、完成态 |
| `warning` | `#FFB74D`（琥珀） | `#FFCC80` | 合规警告、超限提醒（不用红！） |
| `info` | `#64B5F6`（天蓝） | `#90CAF9` | 信息、体重、链接 |
| `background` | `#FAFBFC` | `#121416` | 页面背景 |
| `surface` | `#FFFFFF` | `#1E2124` | 卡片背景 |
| `surfaceElevated` | `#FFFFFF` + shadow | `#2A2D30` | 浮起卡片 |
| `textPrimary` | `#1A1D20` | `#F0F2F5` | 正文 |
| `textSecondary` | `#6B7280` | `#9CA3AF` | 辅助文字 |
| `textTertiary` | `#9CA3AF` | `#6B7280` | 占位、禁用 |
| `divider` | `#E5E7EB` | `#2A2D30` | 分割线 |
| `overlay` | `rgba(0,0,0,0.4)` | `rgba(0,0,0,0.6)` | 模态遮罩 |

#### 数据语义色谱（热量缺口）

```
缺口 ≤ −300 kcal（严重超）  → warning 琥珀 #FFB74D
缺口 −300 ~ 0 kcal（轻超）  → textSecondary 中性灰
缺口 0 ~ +200 kcal（轻亏）  → primary 薄荷 #4ECDC4
缺口 +200 ~ +500 kcal（理想） → success 草绿 #81C784
缺口 > +500 kcal（过大）    → warning 琥珀（安全阀提醒）
```

注意：**超限不用红色**，用琥珀警告；理想区间用绿色，减少心理负担。

#### 运动类型色

| 类型 | 主色 |
|------|------|
| 有氧（跑/骑/游） | `#FF8A65` 珊瑚橘 |
| 力量 | `#7986CB` 薰衣草紫 |
| 柔和（瑜伽/太极/八段锦/气功/五禽戏）| `#81C784` 草绿 |
| 高强度（HIIT/武术）| `#F06292` 樱花粉 |
| 球类/舞蹈 | `#FFB74D` 琥珀 |

### 2.2 字体规范

英文主字体：SF Pro（iOS 系统字体），数字大字使用 SF Pro Rounded 提升亲和力。
中文字体：PingFang SC（iOS 系统）。

| 用途 | 字体 | 字号 | 字重 | 行高 |
|------|------|------|------|------|
| LargeTitle | SF Pro Display | 34pt | Bold 700 | 41pt |
| Title1 | SF Pro Display | 28pt | Bold 700 | 34pt |
| Title2 | SF Pro Display | 22pt | Semibold 600 | 28pt |
| Title3 | SF Pro Display | 20pt | Semibold 600 | 25pt |
| Headline | SF Pro Text | 17pt | Semibold 600 | 22pt |
| Body | SF Pro Text | 17pt | Regular 400 | 22pt |
| Callout | SF Pro Text | 16pt | Regular 400 | 21pt |
| Subheadline | SF Pro Text | 15pt | Regular 400 | 20pt |
| Footnote | SF Pro Text | 13pt | Regular 400 | 18pt |
| Caption1 | SF Pro Text | 12pt | Regular 400 | 16pt |
| Caption2 | SF Pro Text | 11pt | Regular 400 | 13pt |
| **MonoNumber**（大数字展示） | SF Pro Rounded | 44pt / 64pt | Bold 700 | 48pt / 70pt |

支持系统 Dynamic Type，所有字体响应用户系统字号设置。

### 2.3 间距系统（8pt grid）

```
spacing.xxs = 2pt
spacing.xs  = 4pt
spacing.sm  = 8pt
spacing.md  = 12pt
spacing.lg  = 16pt   ← 最常用（卡片内边距、列表行间距）
spacing.xl  = 24pt   ← 卡片间距
spacing.xxl = 32pt   ← 分段间距
spacing.xxxl= 48pt   ← 首页顶部留白
```

### 2.4 圆角与阴影

| Token | 值 | 用途 |
|-------|-----|------|
| `radius.sm` | 8pt | 标签、小按钮 |
| `radius.md` | 12pt | 输入框、列表项 |
| `radius.lg` | 16pt | 卡片 |
| `radius.xl` | 24pt | 大卡片、模态 |
| `radius.full` | 999pt | 圆形（头像、断食环内圈）|
| `shadow.sm` | `0 1 2 rgba(0,0,0,0.04)` | 列表项悬停 |
| `shadow.md` | `0 4 12 rgba(0,0,0,0.06)` | 卡片 |
| `shadow.lg` | `0 8 24 rgba(0,0,0,0.08)` | 浮起（FAB、模态） |

### 2.5 图标规范

- 统一使用 **SF Symbols 5**（iOS 17+ 原生）
- 权重：Regular（正文）/ Semibold（强调）/ Bold（CTA）
- 尺寸：16pt / 20pt / 24pt / 32pt
- 关键业务图标（见下表）

| 业务场景 | SF Symbol |
|---------|-----------|
| 断食 | `timer` / `hourglass` |
| 热量 | `flame.fill` |
| 体重 | `scalemass.fill` |
| 运动 | `figure.run` |
| 计划 | `target` |
| 饮食 | `fork.knife` |
| AI | `sparkles` |
| HealthKit | `heart.fill` |
| iCloud | `icloud.fill` |
| 订阅 | `crown.fill` |

### 2.6 组件规范

#### Button

| 类型 | 样式 | 使用场景 |
|------|------|---------|
| Primary | 填充 primary、白字、高度 52、圆角 12 | 主 CTA（"Start Fasting" / "Log Meal"）|
| Secondary | 描边 primary、primary 字、高度 52 | 次要操作（"Edit Plan"）|
| Tertiary | 无边框 primary 字 | 文本链接 |
| Destructive | 填充 warning（琥珀）、白字 | "Stop Fasting Early" 等非严重破坏性 |
| FAB | 56×56 圆、primary 填充、shadow.lg | 首页右下"+ Log Meal" |

#### Card

- 圆角 16、阴影 md、内边距 16、背景 surface
- Hover 态（iPad）：阴影升级到 lg
- 可点击态：按下时 scale 0.98、阴影变浅，150ms ease-out

#### Input

- 高度 52、圆角 12、1pt 描边 divider
- 聚焦：描边升 2pt primary、背景 surface
- 错误：描边 warning 琥珀 + 下方 warning 文字

---

## 3. 全局导航与框架

### 3.1 TabBar（5 个主 Tab）

```
┌──────────────────────────────────────────┐
│                                          │
│              [CONTENT AREA]              │
│                                          │
├──────────────────────────────────────────┤
│  🏠      🍴     ⏱     📊      ⚙️          │
│ Home   Log   Fast  Stats Settings        │
└──────────────────────────────────────────┘
```

**为什么 5 个 Tab（而不是 4 或 6）**：
- Home 是 Dashboard 总览
- **Log**（饮食）和 **Fast**（断食）是每日最高频两大动作，分别独立 Tab，不藏在二级
- Stats 聚合体重/运动/计划三块（运动/体重/计划不做 Tab 而放 Stats 里的 Segmented Control，避免 Tab 膨胀）
- Settings 单独 Tab，便于找 AI 配置、订阅、iCloud

### 3.2 Tab 详情

| Tab | 图标（SF Symbols）| 默认 View | 对应 PRD §5.1 模块 | 角标规则 |
|-----|-----------------|----------|------------------|---------|
| Home | `house.fill` / `house` | DashboardView | 2. Dashboard + 10. SafetyGuard 横幅 | 无 |
| Log | `fork.knife` | MealLogView（今日饮食）| 3. 饮食记录（L1 Seed / L2 用户 / L3 AI） | 无 |
| Fast | `timer` | FastingView | 4. 断食（8 方案 + Live Activity） | 断食进行中显示绿点 |
| Stats | `chart.bar.fill` | StatsHubView（Segmented: Weight / Workout / Plan）| 5. 体重 + 6. 运动（18 类）+ 7. 计划 | 无 |
| Settings | `gear` | SettingsView | 9. 设置 + 8. Widget 配置入口 + 11. 变现 | AI 未配置显示黄点提醒（首次） |

### 3.3 顶部导航

- 每个 Tab 根页：LargeTitle 导航栏（系统样式），左侧不放返回
- 二级页面：Inline 导航栏，左侧返回 + 居中标题 + 右侧操作按钮
- 模态页面：pageSheet 呈现（iOS 17+ 支持 presentationDetents，可半高/全高）

---

## 4. 首次使用流程 (Onboarding)

**强制 7 屏引导**（合规 + 个性化所需，不可跳过核心屏）：

```
屏 1：欢迎
屏 2：医学免责声明（必须同意）
屏 3：年龄门（必须 ≥ 18）
屏 4：禁用人群自查（孕妇/哺乳/进食障碍/糖尿病等）
屏 5：个人信息（性别/生日/身高/体重/目标体重/活动量）
屏 6：目标与节奏（减脂速度选择）
屏 7：权限（HealthKit / 通知）+ AI 可选配置 + iCloud 可选登录
屏 8：Done，进入 Dashboard
```

### 4.1 屏 1：欢迎

```
┌──────────────────────────────────────────┐
│                                          │
│                                          │
│                  🌿                       │
│              (薄荷色圆形 Logo)             │
│                                          │
│            Welcome to HealthRhythm              │
│                                          │
│     Your clean, honest companion for      │
│      fasting, calories, and weight        │
│                                          │
│                                          │
│                                          │
│                                          │
│                                          │
│          ┌────────────────────┐           │
│          │    Get Started     │           │
│          └────────────────────┘           │
│                                          │
│         Already have an account?          │
│            Restore Purchase               │
│                                          │
└──────────────────────────────────────────┘
```

- 居中 Logo + 标题 + 副标题
- Primary CTA "Get Started"（全宽按钮）
- 底部小字 "Restore Purchase"（订阅恢复入口）
- 点击 CTA → 屏 2（fade-in 300ms）

### 4.2 屏 2：医学免责声明（合规硬屏）

```
┌──────────────────────────────────────────┐
│ ←                                        │
│                                          │
│         Before we begin                   │
│                                          │
│  HealthRhythm is a tracking tool, not medical   │
│  advice. We help you log food, fasting    │
│  windows, and weight — we do not          │
│  diagnose, treat, or cure any condition.  │
│                                          │
│  • Always consult a physician before      │
│    starting any weight loss program       │
│  • Stop using if you feel unwell          │
│  • Not designed for people under 18       │
│                                          │
│  By tapping Continue, you acknowledge     │
│  you have read and understood this        │
│  disclaimer.                              │
│                                          │
│                                          │
│          [ ✓ ] I understand               │
│                                          │
│          ┌────────────────────┐           │
│          │      Continue      │           │
│          └────────────────────┘           │
│                                          │
│          Read full disclaimer →           │
│                                          │
└──────────────────────────────────────────┘
```

- 复选框勾选后"Continue"按钮才可点击
- 勾选状态 + 时间戳写入 `UserProfile.disclaimerAgreedAt`
- "Read full disclaimer →" 打开详细文案页（纯滚动文本）
- **合规要点**：文案严格用 "tool / track / log"，禁用 "cure/treat/medical"

### 4.3 屏 3：年龄门

```
┌──────────────────────────────────────────┐
│ ←                                        │
│                                          │
│         How old are you?                  │
│                                          │
│    Apps like HealthRhythm aren't designed       │
│    for people under 18. We need to        │
│    confirm your age.                      │
│                                          │
│                                          │
│        ┌──────────────────────┐           │
│        │                      │           │
│        │    Date of Birth     │           │
│        │                      │           │
│        │    1990 / 05 / 12    │           │
│        │                      │           │
│        └──────────────────────┘           │
│              (iOS wheel picker)           │
│                                          │
│                                          │
│          ┌────────────────────┐           │
│          │      Continue      │           │
│          └────────────────────┘           │
│                                          │
│                                          │
└──────────────────────────────────────────┘
```

- 轮盘选择 DOB（iOS 原生 DatePicker wheels）
- 提交时计算 age
- **< 18**：弹窗"We're sorry — HealthRhythm isn't designed for people under 18. We hope to see you again when you're ready." + 单一按钮"Exit"直接退出 App
- **≥ 18**：继续屏 4

### 4.4 屏 4：禁用人群自查

```
┌──────────────────────────────────────────┐
│ ←                                        │
│                                          │
│         A quick health check              │
│                                          │
│    Please let us know if any of these     │
│    apply to you. We'll customize your     │
│    experience accordingly.                │
│                                          │
│   ☐  I am pregnant or breastfeeding        │
│   ☐  I have a history of eating disorder   │
│   ☐  I have diabetes or take medication    │
│   ☐  I've had surgery in the past 3 mo.    │
│   ☐  I have a chronic health condition     │
│   ☐  None of the above                     │
│                                          │
│    You can change these answers later     │
│    in Settings.                           │
│                                          │
│          ┌────────────────────┐           │
│          │      Continue      │           │
│          └────────────────────┘           │
│                                          │
└──────────────────────────────────────────┘
```

- 可多选，"None" 和其他互斥
- 勾选任何一个 → 下一步前弹"Please consult your physician"软提示 + 允许继续（不硬拦截）
- 记录结果到 `UserProfile.healthFlags`

#### 4.4.1 各类勾选后的差异化处理（与避坑清单 §3.2 严格对齐）

| 勾选项 | UI 层处理 | 功能禁用 |
|-------|---------|---------|
| `pregnant` 孕哺期 | Home 顶部**常驻琥珀横幅**（见 5.8）；每日目标热量建议值**自动 +300 kcal** | OMAD / 20:4 方案在断食选择器中隐藏 |
| `eatingDisorder` 进食障碍史 | Home 文案去掉 `calorie deficit` 字样，改为中性 `daily target`；**每周一次**温和关怀提示卡 | 所有 > 14 小时断食方案隐藏；激进减脂速度档位（`-0.75 kg/week`）不可选 |
| `diabetes` 糖尿病/服药 | Home 顶部常驻信息横幅"Please follow guidance from your healthcare provider" | 无功能禁用，仅提醒 |
| `recentSurgery` 近期手术 | Settings 顶部出现 **"Pause for now"** 按钮（见 11.3）；Home 顶部建议横幅 | 软建议暂停，但可继续使用 |
| `chronicCondition` 慢性病 | Home 顶部信息横幅 | 无功能禁用 |
| BMI < 18.5（自动计算）| **硬拦截**：跳过目标节奏屏，自动切"Weight maintenance"模式 | 禁用减脂计划生成，体重目标只能 ≥ 当前体重 |

### 4.5 屏 5：个人信息

```
┌──────────────────────────────────────────┐
│ ←                                        │
│                                          │
│         Tell us about you                 │
│                                          │
│  Sex                                      │
│   ○ Female   ● Male   ○ Prefer not say    │
│                                          │
│  Height                                   │
│  ┌────────────┬──────────────┐           │
│  │   175      │     cm ▼     │           │
│  └────────────┴──────────────┘           │
│                                          │
│  Current weight                           │
│  ┌────────────┬──────────────┐           │
│  │   78.5     │     kg ▼     │           │
│  └────────────┴──────────────┘           │
│                                          │
│  Activity level                           │
│  ┌────────────────────────────┐          │
│  │ 🪑 Sedentary (little/no)   │          │
│  │ 🚶 Light (1-3 d/week)     ●│          │
│  │ 🏃 Moderate (3-5 d/week)   │          │
│  │ 💪 Very active (6-7 d/wk)  │          │
│  │ 🔥 Extremely active        │          │
│  └────────────────────────────┘          │
│                                          │
│          ┌────────────────────┐           │
│          │      Continue      │           │
│          └────────────────────┘           │
│                                          │
└──────────────────────────────────────────┘
```

- 单位切换（cm/in, kg/lb）记录到 `UserProfile.unitPreference`
- 美区默认 lb + ft/in，用户可切公制
- 根据身高体重计算 BMI，实时显示在"Current weight"下方（如 `BMI 25.6 — Overweight`）
- **BMI < 18.5**：显示琥珀警告横幅"We noticed your BMI suggests you may already be at a healthy or lower weight. HealthRhythm will focus on weight maintenance, not loss."
- 提交后写入 `UserProfile`

### 4.6 屏 6：目标与节奏

```
┌──────────────────────────────────────────┐
│ ←                                        │
│                                          │
│         What's your goal?                 │
│                                          │
│  Target weight                            │
│  ┌────────────┬──────────────┐           │
│  │   72.0     │     kg ▼     │           │
│  └────────────┴──────────────┘           │
│                                          │
│  That's 6.5 kg to go — great goal!        │
│                                          │
│                                          │
│  How fast do you want to go?              │
│                                          │
│  ┌────────────────────────────────────┐  │
│  │ 🌱 Gentle      -0.25 kg / week    │  │
│  │    Easier to sustain                │  │
│  └────────────────────────────────────┘  │
│  ┌────────────────────────────────────┐  │
│  │ 🎯 Balanced    -0.5 kg / week    ●│  │
│  │    Recommended                      │  │
│  └────────────────────────────────────┘  │
│  ┌────────────────────────────────────┐  │
│  │ 🔥 Aggressive  -0.75 kg / week    │  │
│  │    Requires discipline              │  │
│  └────────────────────────────────────┘  │
│                                          │
│  Estimated timeline: 13 weeks             │
│                                          │
│          ┌────────────────────┐           │
│          │   Create My Plan   │           │
│          └────────────────────┘           │
│                                          │
└──────────────────────────────────────────┘
```

- 实时计算预计周期：`(current − target) × 7700 / (daily_deficit × 7)` 周
- **安全阀**：如果 Aggressive 档位使女性日目标 < 1200 或男性 < 1500，该档位灰出 + 说明 "Not available for your profile — would fall below safe calorie threshold"
- 选完 → 生成 `DietPlan` 写入 SwiftData → 屏 7

### 4.7 屏 7：权限与可选配置

```
┌──────────────────────────────────────────┐
│ ←                                        │
│                                          │
│         Last step                         │
│                                          │
│  ❤️  Apple Health                         │
│     Sync weight, workouts, and           │
│     activity automatically               │
│     [ Enable ]  or  Skip                 │
│                                          │
│  🔔  Notifications                        │
│     Reminders for meals, fasting,         │
│     and weigh-ins                         │
│     [ Enable ]  or  Skip                 │
│                                          │
│  ☁️  iCloud Sync                          │
│     Keep your data in sync across         │
│     your Apple devices                    │
│     [ Enable ]  or  Skip                 │
│                                          │
│  ✨  AI Assistant (Optional)              │
│     Bring your own OpenAI / Claude /      │
│     Gemini key. We don't provide AI.      │
│     [ Configure ]  or  Skip for now      │
│                                          │
│          ┌────────────────────┐           │
│          │       Finish       │           │
│          └────────────────────┘           │
│                                          │
└──────────────────────────────────────────┘
```

- 每个都是 `[Enable]` 按钮 + `Skip` 文字链接
- Tap Enable → 弹 iOS 原生权限请求
- "Configure" → 打开 AI BYOK 配置页（见第 12 节），返回后显示 ✓ 已配置
- "Finish" 永远可点，即使全部 Skip 也能进 App

### 4.8 屏 8：完成动画

```
┌──────────────────────────────────────────┐
│                                          │
│                                          │
│                                          │
│                   ✨                      │
│               (checkmark)                 │
│                                          │
│             You're all set!               │
│                                          │
│        Your plan: 1,820 kcal / day        │
│           Goal: 72.0 kg                   │
│          Timeline: ~13 weeks              │
│                                          │
│                                          │
│                                          │
│          ┌────────────────────┐           │
│          │  Start My Journey  │           │
│          └────────────────────┘           │
│                                          │
└──────────────────────────────────────────┘
```

- 400ms checkmark 绘制动画
- 点击 → 淡入 Dashboard

---

## 5. Dashboard 首页 (HomeView)

### 5.1 页面结构（一屏全景）

```
┌──────────────────────────────────────────┐
│                                          │
│  Good morning, Alex           [🔥 12]    │← 问候 + streak
│  Thursday, Apr 16                        │
│                                          │
│  ╔════════════════════════════════════╗  │
│  ║                                    ║  │
│  ║           1,260 / 1,820            ║  │← 热量杯（大号）
│  ║                                    ║  │
│  ║          ●●●●●●●○○○ (69%)          ║  │
│  ║                                    ║  │
│  ║   Ate 1,260  +  Burned 320         ║  │
│  ║         Budget left: 880           ║  │
│  ║                                    ║  │
│  ╚════════════════════════════════════╝  │
│                                          │
│  ┌─────────────────┐ ┌─────────────────┐ │
│  │      ⏱ 14:23    │ │    ⚖ 78.2 kg   │ │← 断食环 + 体重迷你
│  │                 │ │                │ │
│  │   Fasting...    │ │   ▅▆▇▆▅▄▃▃    │ │
│  │   16:8 · 4h 23m │ │   Down 0.3 kg  │ │
│  │                 │ │   this week    │ │
│  │  [End early]    │ │  [+ Log weight]│ │
│  └─────────────────┘ └─────────────────┘ │
│                                          │
│  TODAY'S TIP                             │
│  ┌────────────────────────────────────┐  │
│  │ 💧 Stay hydrated — water helps     │  │
│  │ manage hunger during fasting.      │  │
│  └────────────────────────────────────┘  │
│                                          │
│  QUICK LOG                               │
│  [🥗 Breakfast] [🍱 Lunch] [🍎 Snack]   │
│                                          │
│                              ┌─────┐    │
│                              │  +  │ FAB │
│                              └─────┘    │
└──────────────────────────────────────────┘
```

### 5.2 热量杯（主 Hero 组件）

- 环形进度：外环代表"今日预算"，从 0 延伸到 `TDEE + burn` 方向
- 中间大数字：已摄入 / 预算（44pt SF Pro Rounded）
- 数字颜色根据缺口状态动态变化（见 2.1 语义色谱）
- 下方两行小字：
  - Line 1：`Ate 1,260 + Burned 320`（带图标，flame + figure.run）
  - Line 2：`Budget left: 880`（或 `Over by 60` 如果超了）
- 点击 → 跳 MealLogView 今日视图
- **达到每日目标摄入下限**（女 1200/男 1500）时，杯底出琥珀小横条"You've reached the minimum calorie floor"

### 5.3 断食卡（左）/ 体重卡（右）

**断食卡（非断食中）：**
```
┌─────────────────┐
│      ⏱          │
│                 │
│  Ready to fast? │
│                 │
│  [Start 16:8]   │
└─────────────────┘
```

**断食卡（进行中）：**
```
┌─────────────────┐
│    ⏱ 14:23     │← 已断食时长（MonoNumber 28pt）
│                 │
│  Fasting...    │
│  16:8 · 4h 23m │← 方案 + 剩余时间
│                 │
│  [End early]   │
└─────────────────┘
```

**体重卡：**
- 右上：今日/最近体重（24pt）
- 中：近 7 日火柴棒图
- 底："Down 0.3 kg this week" 或 "No change"（中性）
- 按钮：`[+ Log weight]`

### 5.4 Today's Tip 卡

- 每日一条，**本地规则引擎驱动**（非 AI，不联网）
- 启动时按**优先级从高到低**逐条匹配，命中即显示，后续规则跳过
- 可右滑 "Dismiss"（当日不再弹），但优先级 1（断食超长警告）**不可 dismiss**

**规则引擎优先级（与 PRD §6.2 对齐）：**

| 优先级 | 条件 | 文案示例 | 可 dismiss |
|-------|------|---------|-----------|
| 1 | 当前断食 ≥ 24h | "⚠️ You've been fasting 24h+ — consider ending" | ✗ 不可 |
| 2 | `eatingDisorder` 用户 + 本周已满 7 天 | "🌿 A gentle check-in — how are you feeling?"（→ §5.8.3 关怀卡）| ✓ |
| 3 | 今日摄入超目标 +500 kcal | "A short walk can rebalance today — [View workouts]" | ✓ |
| 4 | 体重 ≥ 3 天未录 | "Missed your weigh-ins? A quick check helps keep track" | ✓ |
| 5 | 连续 7 天达标（摄入 ≤ 目标） | "🎉 7 days on track — keep going!" | ✓ |
| 6 | 当前断食进行中（< 24h） | "💧 Stay hydrated — water, tea, black coffee are fine" | ✓ |
| 7 | 默认 | 5 条中性 tips 轮播（每日轮换）：<br/>· "Aim for 25g fiber today"<br/>· "Protein at every meal keeps you full"<br/>· "Sleep 7h+ helps weight loss"<br/>· "Walk after meals to aid digestion"<br/>· "Progress is a line, not a point" | ✓ |

**视觉：** 优先级 1 用 `warning` 琥珀背景 + 💛 图标；优先级 2 用 success 绿背景 + 🌿；其他用 `surface` + 对应 emoji。

### 5.5 Quick Log 快捷记餐

- **4 个胶囊按钮**（Breakfast / Lunch / Snack / Dinner），按当前时间智能**突出显示 1–2 个推荐餐次**（primary 填充色），其他餐次显 outline 样式
- 点击任一胶囊 → 直达饮食录入页 + 预填餐次
- **推荐规则（与 §7.1 餐次时间窗口一致）**：

| 当前时间 | 主推（primary 填充） | 次推（outline） |
|---------|-------------------|----------------|
| 5:00–10:30 | Breakfast | Snack |
| 10:30–11:00 | Breakfast / Lunch 并列 | Snack |
| 11:00–14:30 | Lunch | Snack |
| 14:30–17:00 | Snack | Lunch |
| 17:00–21:00 | Dinner | Snack |
| 21:00–5:00 | Snack | Dinner |

### 5.6 FAB 浮动按钮

- 右下 56×56 圆，primary 填充，shadow.lg
- 点击展开 bottom sheet：
  - `+ Log Meal`
  - `+ Log Weight`
  - `+ Log Workout`
  - `Start Fasting`
- iOS 17 用 `Menu` 或自定义 sheet

### 5.7 状态变化

| 场景 | Dashboard 变化 |
|------|--------------|
| 首次进入（无任何数据）| 热量杯为 0，显"Log your first meal to get started"|
| 断食进行中 | 左下角断食卡变成倒计时态（见 5.3）|
| HealthKit 体重刚同步 | 体重卡数字 +toast "Weight updated from Health"|
| 超过每日热量 +500 | Tip 卡显"Consider a walk to balance it out — [View workout ideas]"|
| Pro 用户 | 无广告；免费用户首页底部显横幅广告（尊重距离，不挡核心数据） |
| 孕哺期 | 顶部常驻琥珀横幅（见 5.8）|
| 进食障碍 | Tip 卡每周一次改显关怀卡（见 5.8）；热量杯文案去掉 "deficit" 字样 |

### 5.8 健康人群差异化 UI（合规强制）

#### 5.8.1 孕哺期常驻横幅（Home 顶部，不可关闭）

```
┌──────────────────────────────────────────┐
│ 💛 We care about your safety             │
│ Because you're pregnant or breastfeeding,│
│ please follow guidance from your         │
│ healthcare provider.  [Edit profile →]   │
└──────────────────────────────────────────┘
```

- 琥珀淡色背景（`warning` 20% 透明度）
- 左侧 💛 图标，文字左对齐
- 右侧 `[Edit profile →]` 快捷打开 Health Profile 编辑
- 不提供关闭按钮（常驻），只能通过 Settings 改档案取消
- **自动调整：** 勾选 `pregnant` 或 `breastfeeding` 后，每日目标热量自动 **+300 kcal**（孕哺期额外能量需求，基于 WHO 建议），并切换为**体重维持模式**（不做减脂目标）；Settings → Daily calorie goal 页面显示调整说明"+300 kcal added for pregnancy/breastfeeding"

#### 5.8.2 糖尿病/慢性病/术后：信息横幅（可折叠）

```
┌──────────────────────────────────────────┐
│ ℹ Please follow guidance from your       │
│   healthcare provider.        [×] [Edit] │
└──────────────────────────────────────────┘
```

- 首次展示必现，用户按 × 可隐藏 7 天，到期再出现一次
- 术后用户 × 关闭后 Settings 顶部仍有 "Pause for now"（见 §11.3）

#### 5.8.3 进食障碍关怀卡（每周一次，替代普通 Tip）

```
┌────────────────────────────────────────┐
│ 🌿 A gentle check-in                   │
│                                        │
│ How are you feeling this week? HealthRhythm  │
│ works best as a gentle companion, not  │
│ a source of pressure.                  │
│                                        │
│ [I'm doing well]  [I need a break]     │
└────────────────────────────────────────┘
```

- 按 "I need a break" → 跳 Settings 的 "Pause for now"
- 按 "I'm doing well" → 本周不再弹
- 该人群的**所有文案**不出现 "deficit"、"burn"、"lose"，改用中性 "daily target"、"activity"、"goal"

---

## 6. 断食模块 (FastingView)

### 6.1 主页（FastingHomeView）

**非断食中：**
```
┌──────────────────────────────────────────┐
│ ← Fast                          ⋯        │
│                                          │
│     Last fast: 16:8 · completed ✓        │
│     You've fasted 12 days this month      │
│                                          │
│                                          │
│           ┌─────────────┐                │
│           │             │                │
│           │   PLAN      │                │
│           │             │                │
│           │   16:8 ▼    │                │← 方案选择器
│           │             │                │
│           └─────────────┘                │
│                                          │
│     You'll fast until                    │
│     Thursday, 12:00 PM                   │
│                                          │
│                                          │
│          ┌────────────────────┐           │
│          │    Start Fasting   │           │
│          └────────────────────┘           │
│                                          │
│                                          │
│   This week                               │
│   ● ● ● ○ ● ● ○                          │← streak
│   M  T  W  T  F  S  S                    │
│                                          │
│                                          │
└──────────────────────────────────────────┘
```

**方案选择器（tap 方案下拉）：**
```
┌──────────────────────────────┐
│ Choose a plan                 │
├──────────────────────────────┤
│ 🌱 12:12   Beginner           │
│ 🍃 14:10                      │
│ 🎯 16:8    Recommended  ●    │
│ 🔥 18:6                       │
│ ⚡ 20:4    Advanced           │
│ 💎 OMAD    One meal a day     │
│ 📅 5:2     Weekly             │
│ ✏️  Custom...                 │
├──────────────────────────────┤
│ 🔒 Some plans hidden due to   │ ← 基于健康自查
│    your health profile        │
└──────────────────────────────┘
```

- **方案隐藏规则（与 PRD §6.4 方案表 / 避坑清单 §3.2 严格对齐）：**

| 健康档案 | 隐藏时长类方案 | 隐藏热量限制类方案 | 保留 |
|---------|--------------|-----------------|------|
| `pregnant` / `breastfeeding` | 20:4 / OMAD | — | 12:12, 14:10, 16:8, 18:6, 5:2 |
| `eatingDisorder` | 16:8, 18:6, 20:4, OMAD（所有 > 14h 时长类）| **5:2**（热量限制类，500 kcal/天触发节食行为模式）| 仅 12:12, 14:10 |
| `BMI < 18.5` | 18:6, 20:4, OMAD | — | 12:12, 14:10, 16:8, 5:2 |

- 隐藏逻辑分两类：
  - **时长类隐藏**：按每日断食小时数阈值（避免长时间禁食触发风险）
  - **热量限制类隐藏**：5:2 属于"模拟禁食日 500 kcal"的热量限制模式，对 `eatingDisorder` 用户也触发风险，单独处理
- 被隐藏的方案在列表中仍占位显示但灰出 + 🔒 图标 + 小字说明"Not available for your health profile — [Why?]"
- 自定义方案：设置断食/进食窗口小时数；根据健康档案自动限制最大断食时长（进食障碍用户 ≤ 14h）

**断食进行中：**
```
┌──────────────────────────────────────────┐
│ ← Fast                                   │
│                                          │
│                                          │
│           ╭─────────────╮                │
│         ╱                 ╲              │
│       ╱                     ╲            │
│      │      14:23            │           │ ← 大圆环动画（primary）
│      │                       │           │   中心数字 64pt SF Rounded
│       ╲     fasted          ╱            │
│         ╲    1h 37m left   ╱             │
│           ╰─────────────╯                │
│                                          │
│     Started 10:00 PM · Thu               │
│     Ending 2:00 PM · Fri                 │
│                                          │
│   Progress through zones:                 │
│   🍎 Fed  ●─────○──────○──────○          │
│   🔥 Fat burn  · 12+ hrs                  │
│   ✨ Ketosis   · 14+ hrs                  │
│                                          │
│          ┌────────────────────┐           │
│          │    End Fast        │           │
│          └────────────────────┘           │
│                                          │
│         End fast early →                  │
│                                          │
└──────────────────────────────────────────┘
```

- 大圆环顺时针填充，到进食窗口打开时闪动 + 通知
- Zones 进度条：展示"脂肪燃烧/酮症"等阶段（需标注"Informational only, not medical"）
- "End Fast"（正常结束，绿色完成）vs "End fast early"（琥珀警告文案：会不记入 streak）

#### 6.1.1 断食超长警告（≥ 24 小时，合规强制）

当实际断食时长达到 24 小时时，自动弹出模态阻塞用户当前操作：

```
┌──────────────────────────────────────────┐
│              💛 A heads-up               │
│                                          │
│  You've been fasting for 24 hours.       │
│                                          │
│  Extended fasting (beyond 24h) may       │
│  require medical supervision and is      │
│  not recommended without guidance from   │
│  a healthcare provider.                  │
│                                          │
│  Please consider ending your fast or     │
│  checking in with your doctor.           │
│                                          │
│  [ End fast now ]                        │
│  [ I'm under medical supervision ]       │
│  [ Remind me in 2 hours ]                │
│                                          │
└──────────────────────────────────────────┘
```

- "I'm under medical supervision" → 写入 `SafetyAckLog`（acknowledge 风险）
- "Remind me in 2 hours" → 2 小时后再弹一次
- 达到 36h / 48h 时每次都会再弹，直到用户结束或继续 ack

### 6.2 断食历史（FastingHistoryView）

```
┌──────────────────────────────────────────┐
│ ← History                      ⊕         │
│                                          │
│  This month                               │
│  ┌──────────────────────────────────┐    │
│  │  🔥 12 completed                 │    │
│  │  ⚡ Longest: 18h 12m              │    │
│  │  📈 Avg: 15h 34m                  │    │
│  └──────────────────────────────────┘    │
│                                          │
│  Thursday, Apr 16                         │
│  ● 16:8 · 16h 00m · Completed            │
│                                          │
│  Wednesday, Apr 15                        │
│  ○ 16:8 · 14h 23m · Ended early           │
│                                          │
│  Tuesday, Apr 14                          │
│  ● 16:8 · 16h 02m · Completed            │
│                                          │
│  ...                                      │
│                                          │
└──────────────────────────────────────────┘
```

- ⊕ 按钮：手动补录（选日期 + 开始 + 结束）
- 点击一行 → 详情：可编辑开始/结束时间、备注

---

## 7. 饮食记录模块 (MealView)

### 7.1 今日饮食主页（MealLogView）

```
┌──────────────────────────────────────────┐
│ ← Today                         📅       │
│                                          │
│  Thursday, April 16                      │
│  1,260 / 1,820 kcal  ·  Budget 880        │
│                                          │
│  ┌──────────────────────────────────┐    │
│  │ 🥞 Breakfast          340 kcal   │    │
│  │  • Oatmeal 40g           150 kcal│    │
│  │  • Banana 1 medium       105 kcal│    │
│  │  • Almond milk 250ml      85 kcal│    │
│  │                    [+ Add food]  │    │
│  └──────────────────────────────────┘    │
│                                          │
│  ┌──────────────────────────────────┐    │
│  │ 🍱 Lunch              640 kcal   │    │
│  │  • Grilled chicken       280 kcal│    │
│  │  • Brown rice 150g       165 kcal│    │
│  │  • Mixed salad           195 kcal│    │
│  │                    [+ Add food]  │    │
│  └──────────────────────────────────┘    │
│                                          │
│  ┌──────────────────────────────────┐    │
│  │ 🍎 Snack              280 kcal   │    │
│  │                    [+ Add food]  │    │
│  └──────────────────────────────────┘    │
│                                          │
│  ┌──────────────────────────────────┐    │
│  │ 🍲 Dinner               0 kcal   │    │
│  │                    [+ Add food]  │    │
│  └──────────────────────────────────┘    │
│                                          │
│  Macros                                   │
│  P ███████░░░░░ 80/120g                  │
│  F █████░░░░░░░ 45/60g                   │
│  C ████████░░░░ 150/200g                 │
│                                          │
│  [Copy yesterday's meals]                 │
│                                          │
└──────────────────────────────────────────┘
```

- 📅 按钮：切换日期（默认今日）
- 每餐独立卡片，顺序固定 Breakfast → Lunch → Snack → Dinner，**不允许用户重排**
- 显示小计 + 食物列表
- 长按某条食物 → 编辑/删除/"Add to favorites"
- 宏量（Macros）条：P 蛋白 / F 脂肪 / C 碳水，用户可在设置隐藏
- 底部 "Copy yesterday's meals" 一键复制昨日所有餐次

**餐次时间窗口（用于 Dashboard §5.5 Quick Log 智能推荐 + AI 记餐默认归属 + 推送分发）：**

| 餐次 | 推荐时间窗口 | 说明 |
|------|------------|------|
| 🥞 Breakfast | 5:00–10:30 | 早晨主餐 |
| 🍱 Lunch | 11:00–14:30 | 午间主餐 |
| 🍎 Snack | 14:30–17:00 **或** 20:00–次日 5:00 | 下午加餐 / 夜宵，双窗口 |
| 🍲 Dinner | 17:00–21:00 | 晚间主餐 |

- 时间窗口仅作**推荐 & 默认归属**用途，**用户可随时将任何食物条目手动移到其他餐次**（长按 → Move to）
- AI 记餐解析结果默认按解析时刻的时间窗口归属，用户可改

### 7.2 添加食物页（AddFoodView）

```
┌──────────────────────────────────────────┐
│ ← Add to Breakfast                  ✕    │
│                                          │
│  ┌────────────────────────────────────┐  │
│  │ 🔍 Search foods or scan barcode    │  │
│  └────────────────────────────────────┘  │
│                                          │
│  [ Recent ] [ Favorites ] [ All ]        │← Segmented
│                                          │
│  ✨ Or describe what you ate              │
│  ┌────────────────────────────────────┐  │
│  │ e.g. "Bowl of oatmeal with banana" │  │
│  │                              [Go→] │  │
│  └────────────────────────────────────┘  │
│   (AI-powered · BYOK required)            │
│                                          │
│  Recent                                   │
│  ┌──────────────────────────────────┐    │
│  │ 🥣 Oatmeal 40g       150 kcal  + │    │
│  └──────────────────────────────────┘    │
│  ┌──────────────────────────────────┐    │
│  │ 🍌 Banana 1 medium   105 kcal  + │    │
│  └──────────────────────────────────┘    │
│  ┌──────────────────────────────────┐    │
│  │ 🥚 Egg 1 large        70 kcal  + │    │
│  └──────────────────────────────────┘    │
│  ...                                      │
│                                          │
│            [ Create custom food ]         │
│                                          │
└──────────────────────────────────────────┘
```

- 搜索栏即时搜索（L1 Seed + L2 用户库），键入 300ms debounce
- 未配 AI Key 时"AI describe"区域仍显示但带 lock icon + "Set up AI in Settings"
- 条码扫描（v1.1，MVP 不做，入口按钮禁用 + lock，避免出空壳）
- 搜索结果项：swipe 右 = 加入收藏、swipe 左 = 不再显示

**Free 用户自建食物限制（与 PRD §10.2 对齐）：**

- 免费用户 L2 自建食物库上限 **20 条**；Pro 用户无限
- 达到 18 条（90%）时，`Create custom food` 按钮上方出现轻量 info banner：

```
┌──────────────────────────────────────────┐
│ ⓘ 18 / 20 custom foods used               │
│   Upgrade to Pro for unlimited custom     │
│   foods.                   [ Upgrade → ]  │
└──────────────────────────────────────────┘
```

- 达到 20 条时，`Create custom food` 按钮禁用并显示 🔒 图标，点击弹付费墙（§13.1）
- 已建立的 21+ 条在 Pro 降级后**不删除**，但不可再新建（优雅降级）

### 7.3 食物详情页（份量调整）

```
┌──────────────────────────────────────────┐
│ ← Oatmeal                         ♡      │
│                                          │
│  Per 100g: 389 kcal                       │
│  P 13g · F 7g · C 66g                    │
│                                          │
│  Serving                                  │
│  ┌──────────────────────────────────┐    │
│  │   [ − ]     40 g    [ + ]        │    │
│  └──────────────────────────────────┘    │
│  ○ grams  ● standard (40g)  ○ cup         │
│                                          │
│  Adds to your meal:                       │
│  ┌──────────────────────────────────┐    │
│  │       156 kcal                   │    │
│  │   P 5g · F 3g · C 26g            │    │
│  └──────────────────────────────────┘    │
│                                          │
│  Add to                                   │
│  ● Breakfast ○ Lunch ○ Dinner ○ Snack    │
│                                          │
│          ┌────────────────────┐           │
│          │        Add         │           │
│          └────────────────────┘           │
│                                          │
└──────────────────────────────────────────┘
```

### 7.4 AI 解析页（DescribeMealView）

```
┌──────────────────────────────────────────┐
│ ← AI describe                        ✕   │
│                                          │
│  What did you eat?                        │
│  ┌────────────────────────────────────┐  │
│  │ Bowl of oatmeal with banana and    │  │
│  │ almond milk                        │  │
│  └────────────────────────────────────┘  │
│                                          │
│          [ Analyze with AI ]              │
│                                          │
│  ──────────────────────────────────       │
│                                          │
│  AI found 3 items:                        │
│                                          │
│  ┌──────────────────────────────────┐    │
│  │ ✓ Oatmeal 40g         150 kcal   │    │
│  │   ✎ Edit  ☆ Save to library      │    │
│  └──────────────────────────────────┘    │
│  ┌──────────────────────────────────┐    │
│  │ ✓ Banana 1 medium     105 kcal   │    │
│  └──────────────────────────────────┘    │
│  ┌──────────────────────────────────┐    │
│  │ ✓ Almond milk 250ml    85 kcal   │    │
│  └──────────────────────────────────┘    │
│                                          │
│  Total: 340 kcal · P 10g · F 8g · C 60g  │
│                                          │
│          ┌────────────────────┐           │
│          │   Add all to meal  │           │
│          └────────────────────┘           │
│                                          │
│  ⓘ Estimated by AI. Review before        │
│    logging.                               │
│                                          │
└──────────────────────────────────────────┘
```

- Tap "Save to library" → 入 L2 用户库，下次搜索命中
- 可取消勾选某项不添加
- AI 响应慢时显 skeleton loading
- 错误：显示"We couldn't analyze that — try again or add manually"

---

## 8. 体重记录模块 (WeightView)

**位置：** Stats Tab → Segmented "Weight"

### 8.1 主页

```
┌──────────────────────────────────────────┐
│ ← Stats                                   │
│ [ Weight ] Workout  Plan                 │← Segmented
│                                          │
│       78.2 kg                             │← 当前（MonoNumber 64pt）
│       BMI 25.6 · ↓ 0.3 since last week    │
│                                          │
│  [ 1W ] [ 1M ] [ 3M ] [ 6M ] [ 1Y ] [All]│
│                                          │
│     ┌──────────────────────────────┐     │
│     │                              │     │
│     │   ●─●─                       │     │← 趋势线图
│     │        ╲                     │     │
│     │         ●─●─●                │     │
│     │               ╲              │     │
│     │                ●─●─●         │     │
│     │                      ╲       │     │
│     │                       ●      │     │
│     └──────────────────────────────┘     │
│                                          │
│  Goal: 72.0 kg · 6.2 kg to go             │
│  Est. reach: July 28                      │
│                                          │
│  [ + Log weight today ]                   │
│                                          │
│  Recent entries                           │
│  • Apr 16  78.2 kg   Manual              │
│  • Apr 15  78.4 kg   Health app          │
│  • Apr 13  78.5 kg   Manual              │
│  ...                                      │
│                                          │
└──────────────────────────────────────────┘
```

- **趋势图双线显示（与 PRD §6.5 对齐）：**
  - 实体线：每日体重数据点（低透明度，色 `textSecondary`）
  - **粗实线：7 日移动平均线**（primary 薄荷青）— 减少每日波动噪音，用户感知"真实趋势"
  - **虚线水平**：目标体重线（72.0 kg）
  - 图例（右上小字）：`— Daily   — 7-day avg   --- Goal`
- **单位切换：** 跟随 Settings → Units（kg/lb），数字 + 轴标签 + tooltip + Recent entries 全部同步显示；切换后历史数据自动换算，不需重新录入
- 数据点 tap 后显 tooltip（日期 + 当日体重 + 当日 7 日均值）
- 长按数据点 → 删除/编辑

### 8.2 录入页

```
┌──────────────────────────────────────────┐
│ ← Log weight                        ✕   │
│                                          │
│  Date                                    │
│  ● Today ○ Pick date                     │
│                                          │
│  Weight                                   │
│  ┌──────────────────────────────────┐    │
│  │     78.2       kg ▼              │    │
│  └──────────────────────────────────┘    │
│                                          │
│  Body fat % (optional)                   │
│  ┌──────────────────────────────────┐    │
│  │     22.5       %                  │    │
│  └──────────────────────────────────┘    │
│                                          │
│  Waist (optional)                        │
│  ┌──────────────────────────────────┐    │
│  │     84         cm ▼              │    │
│  └──────────────────────────────────┘    │
│                                          │
│  Note                                     │
│  ┌──────────────────────────────────┐    │
│  │ Weighed before breakfast          │    │
│  └──────────────────────────────────┘    │
│                                          │
│          ┌────────────────────┐           │
│          │        Save        │           │
│          └────────────────────┘           │
│                                          │
│  ☁ This will sync to Apple Health         │
│                                          │
└──────────────────────────────────────────┘
```

---

## 9. 运动记录模块 (WorkoutView)

**位置：** Stats Tab → Segmented "Workout"

### 9.1 主页

```
┌──────────────────────────────────────────┐
│ ← Stats                                   │
│ Weight  [ Workout ]  Plan                │
│                                          │
│  [ Day ] [ Month ] [ Year ]              │← 三档
│                                          │
│  Today · April 16                         │
│                                          │
│        320 kcal                           │← 今日消耗
│        45 min · 2 workouts                │
│                                          │
│  ┌──────────────────────────────────┐    │
│  │                                  │    │
│  │      🏃 Running  60%             │    │← 类型饼图
│  │      💪 Strength 40%             │    │
│  │                                  │    │
│  └──────────────────────────────────┘    │
│                                          │
│  Today's workouts                         │
│  ┌──────────────────────────────────┐    │
│  │ 🏃 Running                        │    │
│  │ 30 min · 5.2 km · 190 kcal       │    │
│  │ Apple Watch · 7:02 AM             │    │
│  └──────────────────────────────────┘    │
│  ┌──────────────────────────────────┐    │
│  │ 💪 Strength training              │    │
│  │ 15 min · moderate · 130 kcal     │    │
│  │ Manual · 6:30 PM                  │    │
│  └──────────────────────────────────┘    │
│                                          │
│  [ + Log workout ]                        │
│                                          │
└──────────────────────────────────────────┘
```

### 9.2 月视图

```
┌──────────────────────────────────────────┐
│  April 2026                               │
│                                          │
│  Total                                    │
│  ┌────────────────────────────────────┐  │
│  │  ⏱ 18h 42m        🔥 8,420 kcal   │  │
│  │  📅 16 active days                 │  │
│  └────────────────────────────────────┘  │
│                                          │
│  By distance                              │
│  🏃 Running     42.3 km                   │
│  🚴 Cycling     88.0 km                   │
│  🚶 Walking    124.5 km                   │
│                                          │
│  Heatmap                                  │
│  ┌──────────────────────────────────┐    │
│  │ ■ ■ ■ ░ ■ ■ ■                    │    │
│  │ ■ ░ ■ ■ ■ ░ ■                    │    │
│  │ ■ ■ ■ ■ ░ ■ ■                    │    │
│  │ ■ ■ ░ ■ ■ ■ ░                    │    │
│  │ ■ ■ ■                             │    │
│  └──────────────────────────────────┘    │
│  Streak: 5 days                          │
│                                          │
│  Types breakdown                          │
│  🏃 Running      42%                      │
│  💪 Strength     25%                      │
│  🧘 Yoga         18%                      │
│  🪷 Tai Chi       8%                      │
│  Others           7%                      │
│                                          │
└──────────────────────────────────────────┘
```

### 9.3 年视图（年度总结风格）

```
┌──────────────────────────────────────────┐
│  2026 so far                              │
│                                          │
│         168.5 km                          │← 年度里程（巨大字号 80pt）
│         total distance                    │
│                                          │
│  ⏱  184 hours active                      │
│  🔥  82,340 kcal burned                   │
│  📅  212 active days                       │
│                                          │
│  ─────────────────────────────────────    │
│                                          │
│  Your longest workout                     │
│  🏃 Running · Mar 23                      │
│  1h 42m · 15.2 km                         │
│                                          │
│  Most loved                               │
│  🏃 Running — 42 sessions                 │
│                                          │
│  You tried 8 activities this year         │
│  🏃 🚴 💪 🧘 🪷 🥋 🏊 🎾                  │
│                                          │
│  [ Share this year ↑ ]                    │← 生成长图分享卡
│                                          │
└──────────────────────────────────────────┘
```

- "Share this year" 生成 1080×1920 分享图（无 before/after 对比，只有数据和图标），可分享到 Messages/社交

### 9.4 录入页（手动快记）

```
┌──────────────────────────────────────────┐
│ ← Log workout                       ✕   │
│                                          │
│  Type                                     │
│  ┌──────────────────────────────────┐    │
│  │ 🏃 Running                        │    │
│  └──────────────────────────────────┘    │
│                                          │
│  (Tap to see all 18 types)               │
│                                          │
│  Duration                                 │
│  ┌──────────────────────────────────┐    │
│  │        30 min                    │    │
│  └──────────────────────────────────┘    │
│                                          │
│  Intensity                                │
│  ○ Low   ● Moderate   ○ High             │
│                                          │
│  Distance (optional)                      │
│  ┌──────────────────────────────────┐    │
│  │     5.2        km ▼              │    │
│  └──────────────────────────────────┘    │
│                                          │
│  Estimated burn                           │
│  ~ 290 kcal                               │← 实时计算，随 Type/Duration/Intensity 变化
│                                          │
│          ┌────────────────────┐           │
│          │        Save        │           │
│          └────────────────────┘           │
│                                          │
│  ⓘ kcal = MET × weight × hours ×         │
│    intensity. Actual burn varies.         │
│                                          │
└──────────────────────────────────────────┘
```

**消耗计算公式（与 PRD §7.5 一致）：**

```
kcal = MET × weight_kg × duration_hours × intensityMultiplier
```

- **MET 值**：来自 §9.5 运动类型表（默认 MET），例如 Running = 8.0
- **intensityMultiplier**：
  - ○ Low → 0.8
  - ● Moderate → 1.0（默认）
  - ○ High → 1.2
- **示例（Alex 82kg 跑步 30 分钟 Moderate）**：`8.0 × 82 × 0.5 × 1.0 = 328 kcal`
- Estimated burn 行在 Type/Duration/Intensity 任一字段变化时**实时更新**，300ms debounce
- 保存后写入 `WorkoutEntry.estimatedKcal`，并同步回 HealthKit `activeEnergyBurned`（若用户开启写回）

### 9.5 运动类型选择器

```
┌──────────────────────────────────────────┐
│ ← Choose activity                    ✕   │
│                                          │
│  🔍 Search                                │
│                                          │
│  Cardio                                   │
│  🏃 Running          🚶 Walking           │
│  🚴 Cycling          🏊 Swimming          │
│  🪁 Jumping rope      🥾 Hiking            │
│  🚣 Rowing                               │
│                                          │
│  Strength & HIIT                          │
│  💪 Strength         🔥 HIIT              │
│  🥋 Martial arts                          │
│                                          │
│  Ball & Dance                             │
│  🏀 Basketball       💃 Dancing           │
│                                          │
│  Mind-Body (Traditional Chinese)          │
│  🧘 Yoga              ☯ Tai Chi           │
│  🪷 Ba Duan Jin      🌬 Qigong           │
│  🦌 Wuqinxi           💃 Square dance     │
│                                          │
└──────────────────────────────────────────┘
```

- 中国传统运动独立分组，中英双名（"Ba Duan Jin" / "八段锦"）
- 搜索支持英文和中文

---

## 10. 减肥计划模块 (PlanView)

**位置：** Stats Tab → Segmented "Plan"

### 10.1 主页

```
┌──────────────────────────────────────────┐
│ ← Stats                                   │
│ Weight  Workout  [ Plan ]                │
│                                          │
│  Your plan                                │
│                                          │
│  ┌──────────────────────────────────┐    │
│  │  Goal                             │    │
│  │  Lose 6.2 kg in 12 weeks          │    │
│  │                                   │    │
│  │  Daily target                     │    │
│  │  1,820 kcal (deficit 500)        │    │
│  │                                   │    │
│  │  Fasting plan                     │    │
│  │  16:8 · daily                     │    │
│  │                                   │    │
│  │  Expected reach                   │    │
│  │  Jul 28 (13 weeks)                │    │
│  └──────────────────────────────────┘    │
│                                          │
│  Progress                                 │
│  ████████░░░░░░░░  47%                   │
│  2.9 kg lost  ·  3.3 kg to go             │
│                                          │
│  Last 4 weeks pace                        │
│  📈 On track (avg −0.5 kg/week)          │
│                                          │
│  [ Edit plan ]  [ Regenerate ]           │
│                                          │
└──────────────────────────────────────────┘
```

### 10.2 重新生成（Regenerate）

- 免费用户每周 1 次（本地配额）
- 超限点击：弹 paywall 或 "Watch ad for one more →"
- 再生时：弹窗 "This will replace your current plan" → 确认 → 回屏 6 节奏页

### 10.3 "如果我..."模拟（v1.1，MVP 不做）

MVP 先不放入口，占位想法（供详细设计文档参考）：
- "如果我每周多走 3 次" → 新时间线
- "如果我每天少吃 100 kcal" → 新时间线

---

## 11. 设置模块 (SettingsView)

### 11.1 主页

```
┌──────────────────────────────────────────┐
│  Settings                                │
│                                          │
│  ┌──────────────────────────────────┐    │
│  │ 👤 Alex Jennings                 │    │
│  │    Member since Apr 2026          │    │
│  │    [ Edit profile ]               │    │
│  └──────────────────────────────────┘    │
│                                          │
│  ACCOUNT                                  │
│  ⭐ HealthRhythm Pro                    Free > │
│  ☁ iCloud Sync                   On   > │
│                                          │
│  PREFERENCES                              │
│  📐 Units (kg, cm)                      > │
│  🌐 Language                   English > │
│  🎨 Appearance                 System  > │
│  🔔 Notifications                       > │
│                                          │
│  HEALTH                                   │
│  ❤️  Apple Health                  On  > │
│  🛡 Health profile                      > │
│  📅 Daily calorie goal    1820 kcal   > │
│  ⏱ Fasting plan             16:8      > │
│                                          │
│  AI (Bring Your Own Key)                  │
│  ✨ AI Provider            Not set    > │
│                                          │
│  PRIVACY                                  │
│  🔒 Do Not Sell My Info (CCPA)       ○── │
│  📊 Personalized ads                  ●─ │
│                                          │
│  DATA                                     │
│  📤 Export data                         > │
│  🗑 Delete all data                     > │
│                                          │
│  ABOUT                                    │
│  📚 Scientific references               > │
│  📜 Privacy policy                       > │
│  📝 Terms of service                     > │
│  🔒 Medical disclaimer                   > │
│  ⓘ About HealthRhythm · v1.0.0                 │
│                                          │
└──────────────────────────────────────────┘
```

**术后用户特别入口（§5.8.2 关联）：**  
`healthFlags` 包含 `recentSurgery` 时，Settings 顶部（账户卡下方）出现一块：
```
┌──────────────────────────────────────────┐
│ 🌿 Recently had surgery?                 │
│    Consider pausing HealthRhythm until you're  │
│    fully recovered.                       │
│    [ Pause for now ]                      │
└──────────────────────────────────────────┘
```
点击 `Pause for now` → 弹确认 → 暂停所有通知 + 首页改显"Resume when you're ready"大按钮 + 记录 `pausedAt` 时间戳。

### 11.2 各二级页要点

- **Edit profile**：性别/身高/体重/活动量/健康自查复选，修改后触发 TDEE 重算
- **Subscription**：跳付费墙或管理订阅（iOS Manage Subscriptions URL）
- **iCloud Sync**：toggle，说明"Your data syncs privately via iCloud"
- **Notifications**：每类推送独立 toggle（饮食/断食/称重/复盘）+ 时段设置
- **Apple Health**：toggle 总开关 + 分类细项（读体重、读 workout、写体重等）；**页面底部常驻说明文字**："Your health data is stored on your device and, if enabled, synced via Apple's end-to-end encrypted iCloud. It is never shared with advertisers or third parties."
- **Health profile**：禁用人群复选框可随时修改（例如用户怀孕了）；修改生效时立即重算 UI 差异化（5.8 节）
- **Do Not Sell My Info (CCPA)**：toggle，开启后在 Privacy Policy 中声明；附说明"We don't sell personal information, but you can opt out of any future data sharing here."
- **Personalized ads**：toggle，控制 ATT 相关的个性化广告；关闭后通过 TopOn 统一关闭个性化追踪，转为 non-personalized ads
- **Language**：English / 简体中文
- **Appearance**：Light / Dark / System
- **Export data**：生成 JSON 或 CSV（饮食/体重/运动/断食），通过 ShareSheet 导出
- **Delete all data**：二次确认 + 输入 "DELETE" 字符串确认；清除本地 + iCloud
- **Scientific references**（简略 wireframe）：

```
┌──────────────────────────────────────────┐
│ ← Scientific References                  │
│                                          │
│  HealthRhythm's calculations and               │
│  recommendations are based on            │
│  peer-reviewed research.                 │
│                                          │
│  ENERGY EXPENDITURE (BMR/TDEE)           │
│  ▸ Mifflin et al. (1990) — AJCN          │
│    A new predictive equation for resting │
│    energy expenditure.                   │
│                                          │
│  BMI CLASSIFICATION                       │
│  ▸ WHO (2004) — The Lancet               │
│    Global Database on BMI.               │
│                                          │
│  INTERMITTENT FASTING                     │
│  ▸ de Cabo & Mattson (2019) — NEJM        │
│    Effects of intermittent fasting on    │
│    health, aging, and disease.           │
│                                          │
│  EXERCISE MET VALUES                      │
│  ▸ Ainsworth et al. (2011) — MSSE        │
│    2011 Compendium of Physical           │
│    Activities.                            │
│                                          │
│  TRADITIONAL CHINESE EXERCISE             │
│  ▸ Xiong et al. (2015) — IJERPH          │
│                                          │
│  ⓘ HealthRhythm is not medical advice.         │
│    Always consult your healthcare        │
│    provider.                              │
│                                          │
└──────────────────────────────────────────┘
```

- **Medical disclaimer**：可重读免责全文

---

## 12. AI BYOK 配置页

```
┌──────────────────────────────────────────┐
│ ← AI Assistant                           │
│                                          │
│  ┌──────────────────────────────────┐    │
│  │ ⓘ HealthRhythm does not provide AI.    │    │
│  │   You bring your own API key     │    │
│  │   from OpenAI, Anthropic, Google │    │
│  │   Gemini, or any compatible      │    │
│  │   service.                       │    │
│  └──────────────────────────────────┘    │
│                                          │
│  Provider                                 │
│  ○ OpenAI (ChatGPT)                      │
│  ● Anthropic (Claude)                    │
│  ○ Google Gemini                         │
│  ○ Custom (OpenAI compatible)            │
│                                          │
│  API Base URL                             │
│  ┌──────────────────────────────────┐    │
│  │ https://api.anthropic.com/v1     │    │
│  └──────────────────────────────────┘    │
│                                          │
│  API Key                                  │
│  ┌──────────────────────────────────┐    │
│  │ sk-ant-••••••••••••••••••  👁    │    │
│  └──────────────────────────────────┘    │
│  ⓘ Stored in Keychain on this device      │
│                                          │
│  Model ID                                 │
│  ┌──────────────────────────────────┐    │
│  │ claude-3-5-sonnet-20241022       │    │
│  └──────────────────────────────────┘    │
│   Suggestions:                            │
│   · claude-3-5-sonnet-20241022           │
│   · claude-3-5-haiku-20241022            │
│   · claude-3-opus-20240229               │
│                                          │
│  Advanced (optional)                      │
│  Temperature: 0.3   Max tokens: 2000     │
│                                          │
│          ┌────────────────────┐           │
│          │   Test Connection  │           │
│          └────────────────────┘           │
│                                          │
│          ┌────────────────────┐           │
│          │        Save        │           │
│          └────────────────────┘           │
│                                          │
└──────────────────────────────────────────┘
```

- Key 字段默认掩码，眼睛图标切换明文
- Test Connection：发一个极小 prompt "reply with 'ok'"，成功显 ✓ 绿勾 + 响应时长（如 "245 ms"）

**Test Connection 4 类错误分类识别（与 PRD §6.9 对齐）：**

| 错误类型 | HTTP / 条件 | UI 提示 | 建议操作 |
|---------|------------|--------|---------|
| 网络错误 | URLError / timeout | ❌ "Can't reach server" | Check your internet connection; verify Base URL is correct |
| 鉴权失败 | 401 / 403 | ❌ "Invalid API key" | Double-check your key; make sure it hasn't been revoked |
| 配额超限 | 429 / 402 | ❌ "Quota exceeded" | Check your billing / usage at provider dashboard |
| 模型不存在 | 404 / `model_not_found` | ❌ "Model not available" | Verify Model ID; see suggestions list |
| 其他 | — | ❌ "Unknown error: {code}" | Report to support |

**保存后激活的 AI 功能入口：**

| 功能 | 版本 | 入口位置 |
|------|------|---------|
| 🗣 Describe what you ate（自然语言记餐解析）| v1.0 MVP | §7.2 添加食物页顶部 |
| 📊 Weekly diet review（饮食周报诊断） | v1.0.x | §10.1 计划主页底部 / §11.1 Settings |
| 🔄 Food replacement suggestions（食物替换建议） | v1.0.x | §7.3 食物详情页底部按钮 |
| 🎯 Plan tuning suggestions（计划调优） | v1.1 | §10.1 计划主页 |

- 未保存有效配置前，以上入口全部显示 🔒 图标 + "Set up AI in Settings"
- 用户可在任一入口处 tap 🔒 → 直达 §12 AI 配置页

---

## 13. 付费墙 (Paywall)

### 13.1 主付费墙

```
┌──────────────────────────────────────────┐
│                                      ✕  │
│                                          │
│             ⭐ HealthRhythm Pro                 │
│                                          │
│     Unlock your full weight loss         │
│            companion                      │
│                                          │
│  ✓ No ads                                 │
│  ✓ Multiple plan scenarios                │
│  ✓ Advanced reports (month/year)          │
│  ✓ AI assistant access (BYOK)            │
│  ✓ All widgets & live activities          │
│  ✓ Unlimited custom foods                 │
│  ✓ Data export                            │
│                                          │
│  ┌──────────────────────────────────┐    │
│  │                                  │    │
│  │        $12.99 / year             │    │← Annual highlighted
│  │        $1.08 / month              │    │
│  │       7-day free trial           ●│    │
│  │                                  │    │
│  └──────────────────────────────────┘    │
│                                          │
│  ┌──────────────────────────────────┐    │
│  │        $1.99 / month              │    │
│  │       No commitment               │    │
│  └──────────────────────────────────┘    │
│                                          │
│          ┌────────────────────┐           │
│          │  Start free trial  │           │
│          └────────────────────┘           │
│                                          │
│  Auto-renewable subscription.             │
│  Subscription automatically renews        │
│  unless canceled at least 24 hours        │
│  before the end of the current period.    │
│  Cancel anytime in Settings.              │
│                                          │
│    Terms · Privacy · Restore Purchase     │
│                                          │
└──────────────────────────────────────────┘
```

**合规要点（硬性，与避坑清单 §7.2 逐项对齐）：**
- ✅ 显示 `Auto-renewable subscription` 文字
- ✅ 显示 `Subscription automatically renews unless canceled at least 24 hours before the end of the current period`（App Store 强制要求原文）
- ✅ 显示 `Cancel anytime in Settings`
- ✅ 每档价格来自 StoreKit `Product.displayPrice`，不硬编码
- ✅ 试用档显示 `7-day free trial, then $12.99/year`
- ✅ Restore Purchase 入口
- ✅ Terms / Privacy 可点击（打开 SFSafariViewController）
- ✅ 关闭按钮（X 左上，44×44 点击区域）
- ❌ 不做首启立即弹（让用户先体验 3 天）
- ❌ 不暗示 "save X%" 如果不真实
- ❌ 不用 "Free!" 单字误导

### 13.2 触发时机

- 首次进入 Settings → Pro 行
- 免费用户点锁功能时（如 AI 入口、多计划对比）
- 首次记录满 3 天后 Home 底部出"Try Pro"卡片（非强推）

---

## 14. 合规弹窗与警告

### 14.1 每日热量下限警告

```
┌──────────────────────────────────────────┐
│                                          │
│           ⚠  A gentle heads-up           │
│                                          │
│  Your daily calorie goal (1,100 kcal) is │
│  below the safe minimum recommended by    │
│  health experts (1,200 kcal for women).  │
│                                          │
│  Very low intake can affect your energy,  │
│  mood, and metabolism. Please consider    │
│  a gentler pace.                          │
│                                          │
│  [ Use 1,200 instead ]                    │
│  [ I understand the risks, keep 1,100 ]   │
│                                          │
└──────────────────────────────────────────┘
```

- 琥珀色图标，不用红色
- "I understand" 需要点击才关闭
- 事件记录：`risk_acknowledgment` 本地日志

### 14.2 BMI 过低拦截

```
┌──────────────────────────────────────────┐
│                                          │
│           💛 We care about you           │
│                                          │
│  Based on your height and weight, your    │
│  BMI is 17.2, which is below the healthy  │
│  range.                                   │
│                                          │
│  HealthRhythm is designed for weight loss or    │
│  maintenance, not for further reduction   │
│  at your current weight.                  │
│                                          │
│  We'll switch you to weight maintenance   │
│  mode. You can always speak with a        │
│  healthcare provider for personalized     │
│  guidance.                                │
│                                          │
│          ┌────────────────────┐           │
│          │    OK, got it      │           │
│          └────────────────────┘           │
│                                          │
└──────────────────────────────────────────┘
```

### 14.3 未成年人硬拦截

```
┌──────────────────────────────────────────┐
│                                          │
│           💛 Thanks for your interest    │
│                                          │
│  HealthRhythm isn't designed for people under  │
│  18. Nutrition and fasting needs are     │
│  different while you're growing.         │
│                                          │
│  We hope to see you when you're ready.   │
│                                          │
│          ┌────────────────────┐           │
│          │       Exit         │           │
│          └────────────────────┘           │
│                                          │
└──────────────────────────────────────────┘
```

- 点 Exit → App 关闭（iOS 不允许直接 kill，引导用户按 home）
- 或显示一张静态墙，拒绝进入主界面，只有"Change age"入口

### 14.4 ATT Pre-prompt（广告追踪前置解释屏）

iOS 14.5+ 的 `ATTrackingManager.requestTrackingAuthorization` 系统弹窗之前，先显示我们自己的解释屏（提升同意率 + 符合 Apple "不误导"要求）：

```
┌──────────────────────────────────────────┐
│                                     ✕  │
│                                          │
│              🎯                           │
│                                          │
│       Keep HealthRhythm free with              │
│          personalized ads                 │
│                                          │
│  Allowing tracking helps us show you     │
│  more relevant ads and keeps HealthRhythm      │
│  free.                                    │
│                                          │
│  ✓ Your health data is NEVER shared      │
│    with advertisers                       │
│  ✓ You can change this anytime in        │
│    Settings                               │
│                                          │
│                                          │
│          ┌────────────────────┐           │
│          │       Allow        │           │
│          └────────────────────┘           │
│                                          │
│          ┌────────────────────┐           │
│          │      Not now       │           │
│          └────────────────────┘           │
│                                          │
└──────────────────────────────────────────┘
```

- 触发时机：首次启动完成 Onboarding 后、第一次看到广告前（不在 Onboarding 内触发）
- 按 "Allow" → 触发 iOS 系统 ATT 弹窗
- 按 "Not now" → **不触发**系统弹窗（用户保留后续同意机会），此次不做个性化广告
- 关键合规点：**不论同意还是拒绝**，App 所有功能必须完全可用
- 点击 ✕ 等同于 "Not now"

### 14.5 合规弹窗设计总原则

| 场景 | 色调 | 图标 | 语气 |
|------|------|------|------|
| 健康风险警告 | 琥珀（`warning`）| 💛 | 关怀 > 恐吓 |
| 拦截性错误 | 琥珀 | ⚠ | 解释 + 可行建议 |
| 年龄门拦截 | 中性 | 🌿 | 友善告别 |
| ATT / 权限请求 | 品牌色 | 图标对应场景 | 透明 + 利益说明 |

**绝对不用**红色做合规警告，避免用户产生羞耻或焦虑。

---

## 15. Widget 与 Live Activity

### 15.1 主屏 Widget

#### Small（2×2）

```
┌─────────────┐
│    ⏱ 14:23 │← 大数字（MonoNumber）
│             │
│  Fasting...│
│  1h 37m left│
└─────────────┘
```

或（非断食）：

```
┌─────────────┐
│  1260/1820  │
│             │
│   ●●●●○○○   │
│             │
│   69% today │
└─────────────┘
```

#### Medium（4×2）

```
┌──────────────────────────────┐
│  ⏱ Fasting 14:23             │
│  ●●●●●●●●○○○ 1h 37m left    │
│                              │
│  🔥 1,260/1,820 kcal         │
│  ⚖ 78.2 kg ↓ 0.3 this week  │
└──────────────────────────────┘
```

#### Large（4×4）

包含 Medium + 近 7 日体重迷你图 + 本周运动 streak

### 15.2 锁屏 Widget（iOS 16+）

- Circular：断食进度圆环
- Inline：`14:23 fasting`
- Rectangular：`14h 23m · 1h 37m left`

### 15.3 Live Activity（iOS 17+）

**动态岛 紧凑态（左）：**
```
(⏱ 14:23)
```

**动态岛 紧凑态（右）：**
```
(1h 37m)
```

**动态岛 扩展态：**
```
┌─────────────────────────────────┐
│  ⏱ Fasting 16:8                │
│                                 │
│  ●●●●●●●●●○○○  14h 23m / 16h   │
│                                 │
│  Ends at 2:00 PM                │
│                                 │
│  [ End Fast ]                   │
└─────────────────────────────────┘
```

**锁屏：**
```
┌─────────────────────────────────┐
│  🌿 HealthRhythm        Fasting 16:8  │
│                                 │
│            14:23                │
│          14h 23m fasted         │
│                                 │
│  ●●●●●●●●●○○○  Ends 2:00 PM    │
└─────────────────────────────────┘
```

- 断食开始自动启动 Live Activity
- 断食结束后 60 秒停留展示"Completed ✓ 16h 02m" 然后消失
- 超过 iOS 限制（8 小时后）自动转为推送通知定期更新

### 15.4 互动 Widget（iOS 17+）

- Small/Medium Widget 支持"End Fast"按钮
- Medium Widget "+Log breakfast"快捷（带预设早餐时间）
- 要求配置 App Intents

---

## 16. 动画与微交互

### 16.1 原则

- 所有动画 ≤ 400ms
- 主要用 `ease-out` 和 `spring(response:0.4, dampingFraction:0.8)`
- 避免炫技，聚焦反馈性

### 16.2 关键动画

| 场景 | 动画 | 时长 |
|------|------|------|
| Tab 切换 | Fade + 微位移 8pt | 200ms ease-out |
| 卡片出现 | Fade + scale 0.96→1.0 | 250ms spring |
| 按钮按下 | scale 1.0→0.96 | 100ms ease-out |
| 热量杯数值变化 | 数字 tick 动画 + 环形填充 | 400ms spring |
| 断食开始 | 圆环从细到粗 + 粒子涌入 | 600ms |
| 断食完成 | 圆环完成闪光 + haptic success | 800ms |
| 目标达成 | 全屏礼花（粒子）+ haptic notification | 1200ms |
| 删除记录 | swipe → fade + 高度塌缩 | 300ms |
| toast 出现 | 底部滑入 | 200ms ease-out |

### 16.3 触觉反馈（Haptics）

- 按钮 tap：`UIImpactFeedbackGenerator .light`
- 确认操作：`.medium`
- 完成断食/达成目标：`UINotificationFeedbackGenerator.success`
- 警告出现：`.warning`
- 错误：`.error`

---

## 17. 暗色模式适配

- 所有色彩使用 Asset Catalog 动态色（Light/Dark 自动切换）
- 图表线条颜色在 Dark 模式下提亮 10%
- 阴影在 Dark 模式下减弱（避免黑上加黑）
- 图片资源（Logo、空态插画）提供双模式版本
- 用户可在设置强制选择（Light / Dark / System）

---

## 18. iPad 适配

### 18.1 布局策略

- iPad 在 Regular × Regular 尺寸用 `NavigationSplitView` 三列布局：
  - 左：主 Tab 列表
  - 中：二级列表（如饮食记录列表）
  - 右：详情（如食物详情）
- 首页 Dashboard 在 iPad 采用 2×2 网格布局（热量杯 / 断食卡 / 体重卡 / 运动卡）
- Widget 支持 iPadOS 所有尺寸

### 18.2 键盘支持

- 列表页 ↑↓ 键导航，Enter 选中
- Cmd+N 新建记录
- Cmd+, 打开设置

---

## 19. 无障碍设计

- 所有交互元素 VoiceOver 标签完备
- Dynamic Type 全部响应（最大 AX5 可用）
- 最小点击区域 44×44
- 色彩对比度 WCAG AA（正文 4.5:1、大字 3:1）
- 不仅靠颜色传递信息（热量缺口同时有数字 + 图标 + 文字）
- Reduce Motion 开启时所有动画降级为 fade
- 支持 Siri Shortcuts："Log my breakfast" / "Start fasting"（v1.1）

---

## 20. 空态/加载/错误

### 20.1 空态

每个列表页都有友好的空态插画 + 文案 + 行动按钮：

- 饮食空态："No meals logged yet today — [Add your first meal]"
- 体重空态："Start tracking your progress — [Log weight]"
- 运动空态："Your workouts will show up here. Connect Apple Health or log manually — [Enable Health] [Log workout]"
- 断食历史空态："No fasting sessions yet — [Start fasting]"

### 20.2 加载

- 首次启动：Launch Screen + 400ms 加载进 Dashboard
- 数据拉取：skeleton loading（灰色卡片骨架）
- AI 请求：进度条 + "Thinking..." 文字（超 8 秒显示"Still working..."）

### 20.3 错误

- 网络错误（AI）：顶部 toast "Couldn't reach AI — try again later"
- HealthKit 权限拒绝：二级页面内 banner "Health access disabled — Enable in Settings"
- iCloud 同步失败：设置页 iCloud 行显黄点 + "Last synced 2h ago · tap to retry"
- 致命错误（数据损坏）：全屏错误页 + "Contact support" 邮件链接

---

## 附录 A：页面路由全景图

```
Root
├── Onboarding (未完成时 mandatory)
│   ├── Welcome
│   ├── Disclaimer
│   ├── AgeGate
│   ├── HealthCheck
│   ├── PersonalInfo
│   ├── Goal
│   └── Permissions
│
└── MainTabView
    ├── Home (Dashboard)
    │   └── [FAB] → QuickAdd sheet
    │
    ├── Log (Meal)
    │   ├── MealLogView
    │   ├── AddFoodView
    │   │   ├── FoodDetailView
    │   │   ├── DescribeMealView (AI)
    │   │   └── CreateCustomFoodView
    │   └── CopyPreviousDayView
    │
    ├── Fast
    │   ├── FastingHomeView
    │   ├── FastingActiveView (active)
    │   ├── FastingHistoryView
    │   └── FastingDetailView
    │
    ├── Stats
    │   ├── Segmented: Weight | Workout | Plan
    │   ├── WeightView
    │   │   ├── WeightLogView
    │   │   └── WeightDetailView
    │   ├── WorkoutView
    │   │   ├── WorkoutLogView
    │   │   ├── WorkoutTypePicker
    │   │   ├── DayView / MonthView / YearView
    │   │   └── YearShareView
    │   └── PlanView
    │       └── EditPlanView
    │
    └── Settings
        ├── EditProfileView
        ├── SubscriptionView (Paywall)
        ├── iCloudSyncView
        ├── NotificationsView
        ├── AppleHealthView
        ├── HealthProfileView
        ├── AIConfigView (BYOK)
        ├── ExportDataView
        ├── DeleteDataView
        ├── ScientificReferencesView
        ├── PrivacyPolicyView
        ├── TermsView
        └── DisclaimerView
```

---

## 附录 B：设计交付清单（给视觉设计师）

- [ ] App 图标（1024×1024 + 全尺寸套）
- [ ] Logo（矢量 + PNG）
- [ ] 品牌色 Asset Catalog 文件
- [ ] 空态插画 7 张（饮食/体重/运动/断食/计划/搜索/网络）
- [ ] 年度总结卡模板（1080×1920）
- [ ] 付费墙英雄图
- [ ] 首启屏品牌图
- [ ] Widget 预览（3 种尺寸 ×2 模式）
- [ ] App Store 截图（6 张 + 英文 + 简中）
- [ ] 合规弹窗配图（琥珀警告 / 爱心告知）

---

**文档结束 · v1.0**
