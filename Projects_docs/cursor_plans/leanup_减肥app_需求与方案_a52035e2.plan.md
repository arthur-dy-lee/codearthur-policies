---
name: LeanUp 减肥App 需求与方案
overview: 综合型 MVP 减肥 App：以"轻断食计时 + 饮食热量记录 + 体重曲线"为三大核心，叠加可选的 BYOK AI 能力（用户自带模型 URL/Key），免费 + 激励广告 + 订阅去广告的混合变现模式，沿用 FireUp/InnerRelease 的 SwiftUI + SwiftData 架构和 Shared 共享包。
todos:
  - id: prd
    content: 写 Projects/LeanUp_App/docs/LeanUp_PRD_v1.0.md（完整产品需求文档，按 FireUp 风格，含运动模块与合规章节）
    status: pending
  - id: tech-design
    content: 写 Projects/LeanUp_App/docs/LeanUp_详细设计.md（SwiftData 模型含 WorkoutEntry/Type、TDEE/MET 算法、BYOK AI 服务层、HealthKit 双通道、SafetyGuard、食物库 seed 构建脚本、Widget）
    status: pending
  - id: ui-spec
    content: 写 Projects/LeanUp_App/docs/LeanUp_UI设计与交互.md（Dashboard、断食环、记餐流、体重曲线、运动记录、日月年汇总图、计划问卷、首启引导三屏、设置页、付费墙 wireframe）
    status: completed
  - id: compliance
    content: 写 Projects/LeanUp_App/docs/LeanUp_避坑清单.md（Apple 2026 审核对标：10 条硬性红线 + 6 类禁用人群引导流程 + 地雷词清单 + QA 用例 + Review Note 模板 + GDPR/CCPA 合规）
    status: completed
  - id: tasks
    content: 写 Projects/LeanUp_App/tasks.yaml（MVP 分 6 sprint 的 matrixAppsCore 任务编排）
    status: pending
  - id: confirm
    content: 按用户反馈调整名字、市场、Widget、iCloud 等待确认项
    status: pending
isProject: false
---


## 一、产品定位与命名

**产品名：LeanUp — Fasting · Calories · Weight**（已锁定）
- 英文主语言，副标题 `Fasting · Calories · Weight` 或 `Your Weight Loss Companion`
- 系列感呼应 `FireUp`（Up 系列）
- 目标市场：**纯海外**（美欧英加澳 + 东南亚），首发 App Store 美区
- 语言：英文主 + 简体中文次（面向海外华人），其他语言 v1.1
- 一句话：**让减肥像攒钱一样看得见进度** —— 打开 App 立刻看到"本周热量缺口 −3400 kcal / 已持续轻断食 14 天 / 距目标体重还差 4.2 kg"。
- 目标用户：25–40 岁有减脂诉求但不想请教练/不愿严格健身的海外都市人群，女性轻度减脂 + 男性体重管理。
- 差异化定位：市面上 Simple / Zero 只做断食，MyFitnessPal 只做热量，Lifesum 偏综合但笨重；我们把"断食 × 热量 × 体重 × 运动"四件套用同一条进度条串起来，低价 $1.99 切入主流 $9.99–$19.99 市场。

---

## 二、核心价值（JTBD）

> 当我想减肥但又怕太累放弃时，我需要一个能同时帮我"定计划 + 记录 + 看到进度"的工具，让我每天只花 30 秒就能知道今天该吃多少、该不该开吃、体重在不在预期轨道上，以便我持续坚持并看到结果。

三大情绪价值：
- **确定性**：AI（或规则引擎）根据身高/体重/目标/活动量给出每日 TDEE 与建议缺口，告诉你"再坚持 X 天能到目标"
- **低门槛**：饮食只记高频项，不做高精度营养计算，3 秒记一餐
- **可视化进度**：体重曲线 + 断食环 + 热量杯 三位一体

---

## 三、MVP 功能模块（v1.0）

| 模块 | 功能要点 | 优先级 |
|------|---------|--------|
| 首页 Dashboard | 今日热量杯（已摄入/TDEE/缺口）、断食环（进行中/剩余时间）、体重趋势迷你图、今日建议卡 | P0 |
| 断食计时 | 预设方案（16:8 / 18:6 / 20:4 / 5:2 / OMAD / 自定义）、开始/暂停/结束、推送提醒、历史记录、连续打卡（streak） | P0 |
| 饮食记录 | 早/中/晚/加餐四栏；四层录入方式：① 内置 Seed 食物库搜索（500–800 条高频中式 + 快餐连锁，离线可用）② 用户自建食物库（长期积累个人常吃）③ 常用收藏/最近记录 ④ AI 自然语言录入（BYOK 可选，支持"一碗红烧肉"式口语解析 + 一键入库）；支持份量调整、一键复制昨日 | P0 |
| 体重记录 | 每日/每周录入，趋势图（7/30/90/全部），BMI、体脂（手动）、腰围可选，对接 HealthKit 读写 | P0 |
| **运动能量记录** | 12+ 类常见运动（跑步/快走/骑行/力量/瑜伽/HIIT/游泳/跳绳/徒步/舞蹈）**+ 中国传统运动（八段锦/太极/气功/五禽戏/广场舞）**；三种录入：① HealthKit 自动读取 Workouts（Apple Watch / Strava / Keep / Nike 等任何写入 HK 的 App）② 手动输入（类型 + 时长 + 强度，按 MET 估算消耗）③ 运动汇总图（日/月/年三维度：里程 / 时长 / 消耗 / streak）；**不做**动作库/训练计划/视频教程/社交打卡/私教对话；**不做**内置 GPS（v1.1 再评估，MVP 依赖 HealthKit 读其他 App 已记录的距离/心率/路径） | P0 |
| 减肥计划 | 新手问卷（身高/体重/目标/活动量/偏好）→ 生成计划：TDEE、每日目标热量、断食方案推荐、预计达成周期；支持手动调参与重新生成；**热量缺口公式 = TDEE + 今日运动消耗 − 已摄入** | P0 |
| 统计与报告 | 周报/月报：平均摄入、缺口累计、体重变化、断食完成率、运动总里程/总时长、"坚持了 X 天"成就 | P1 |
| 提醒 & Widget & Live Activity | ① 推送提醒：饮食、断食开始/结束、称重、每日复盘；② 桌面 Widget（小/中/大三档）：断食环、今日热量杯、体重迷你图、今日运动消耗；③ 锁屏 Widget：断食剩余小时、热量缺口；④ **Live Activity**：断食进行中的动态岛 + 锁屏实时倒计时（杀手级匹配断食体验）；⑤ 互动 Widget：桌面直接"+记一餐 / 结束断食"按钮 | P0 |
| AI 助手（可选，BYOK） | 自然语言记餐、本周饮食诊断、替换建议、文本问答。**不提供任何内置模型**，全部需用户填 URL/Key | P1 |
| 设置 | 单位（公制/英制）、主题、隐私、iCloud 同步、AI 平台配置、订阅管理、数据导出 | P0 |

v1.1 候选（MVP 后第一优先级）：内置 GPS 运动追踪（跑步/骑行路径地图）、Apple Watch App、iCloud 同步  
v2.0 候选：喝水打卡、食物拍照识别、社群/打卡圈、睡眠整合

---

## 四、AI 能力（BYOK 架构，与 FireUp 一致）

用户在 `设置 → AI 服务` 配置，App 本身不代理任何模型：

- **平台选项**：ChatGPT / Gemini / Claude / 自定义（OpenAI 兼容 API）
- **必填**：Base URL、API Key、模型 ID（手填，支持平台新模型无需 App 更新）
- **可选**：温度、最大 tokens
- **存储**：本地 Keychain，不上传云端；提供"测试连接"按钮
- **场景**（全部可降级为纯规则/本地库）：
  1. **自然语言记餐**："我中午吃了一碗牛肉面和一个鸡蛋" → 结构化 JSON（食物、份量、估算热量 + 蛋/脂/碳），用户确认后入账，并可"保存到我的食物库"下次直接搜
  2. **食物库兜底**：内置 Seed 库搜不到的食物 → AI 估算营养数据 → 用户入库
  3. **饮食诊断周报**：分析一周饮食结构，给出 2–3 条改善建议
  4. **食物替换建议**："想吃炸鸡但在减脂" → 3 个低卡替代
  5. **计划调优**：根据近 7 天执行情况，微调每日目标
- **无 Key 时行为**：所有 AI 入口折叠或显示"前往设置接入你的 AI"，核心记录/计时功能完全可离线使用。
- 技术复用 FireUp 的 `AIServiceProvider` 协议 + 多平台适配（见 [Projects/FireUp_App/docs/FIRE_产品需求文档_PRD.md](Projects/FireUp_App/docs/FIRE_产品需求文档_PRD.md) 第 419–466 行）。

---

## 五、食物库策略（三层架构）

| 层级 | 数据来源 | 离线 | 规模 | 说明 |
|------|---------|------|------|------|
| L1 Seed 内置库 | **USDA FoodData Central**（美国农业部公开数据库，100 万+ 条）精选 + 美英主流快餐连锁（麦当劳/肯德基/星巴克/Subway/Chipotle/Tim Hortons/Panera）+ 常见中式外卖（面向海外华人）| 是 | 1000–1500 条 | JSON seed 打包进 App，首次启动导入 SwiftData，英文 + 中文别名双字段 |
| L2 用户自建库 | 用户手动新建 or AI 解析后入库 | 是 | 无限 | SwiftData 存储，iCloud 可选同步 |
| L3 AI 解析 | BYOK，用户自配模型 | 否（需网络） | — | 自然语言 → 结构化；支持"保存到我的库"沉淀 |

**Seed 构建方案**（不用人肉录入）：
- 用 Python 脚本从 **USDA FoodData Central 开放 API / CSV 下载**拉取数据，筛选高频 1000 条清洗成 `food_seed.json`（字段：name_en, name_zh, aliases, kcal_per_100g, protein_g, fat_g, carb_g, fiber_g, serving_unit, serving_size, category, source）
- 快餐连锁单独一个 `brand_seed.json`，来源：各连锁官网营养标签页面爬取，带 brand 字段
- 中式外卖补充 200 条（炒饭/炒面/dim sum 等海外华人高频）
- 放在 `ios_workspace/LeanUp/Resources/`，App 首次启动 `FoodLibraryBootstrap` 导入
- 详细脚本写在"详细设计"文档里

**搜索体验**：搜索框优先 L2（个人库）→ L1（内置）→ 显示"未找到？用 AI 解析"按钮触发 L3（需 BYOK）。

---

## 六、运动能量记录模块（定位：减脂闭环必备，不是训练 App）

### 6.1 设计红线

| 做 | 不做 |
|----|------|
| 估算/读取"今天消耗了多少 kcal、多少公里、多少分钟" | 动作库 / 训练计划 / 视频教程 |
| 日/月/年三维度汇总图 | 社交打卡圈 / 排行榜 |
| HealthKit Workouts 自动同步 | 私教 AI 对话 / 训练指导 |
| 手动快速记录兜底 | 内置 GPS 追踪（v1.1 评估） |

### 6.2 运动类型清单（MVP 首发 18 种）

**常规运动**：跑步、快走、骑行、力量训练、瑜伽、HIIT、游泳、跳绳、篮球、舞蹈、徒步、划船机  
**中国传统养生运动**：八段锦（MET 2.5）、太极（MET 3.0）、气功（MET 2.5）、五禽戏（MET 2.5）、广场舞（MET 4.0）、武术（MET 5.0）  

每种运动绑定：图标、默认 MET 值、支持的录入方式、推荐强度档位（低/中/高 = MET × 系数）

### 6.3 三种录入路径

**路径 A：HealthKit 自动同步（推荐，0 操作）**
- 读取 `HKWorkout`：运动类型、开始/结束时间、时长、总消耗、距离（跑步/骑行有）、平均心率、最大心率
- 数据来源兼容：Apple Watch 原生 / Strava / Nike Run Club / Keep / 咕咚 / Runkeeper 等所有写 HealthKit 的 App
- App 每次启动拉取最近 30 天 Workouts，新数据自动入账，去重（按 UUID）

**路径 B：手动快记（无手表兜底）**
- 用户选类型 + 时长（分钟）+ 强度（低/中/高）
- 公式：`消耗 kcal = MET × 体重 kg × 时长 h × 强度系数`
- 跑步/骑行/徒步额外可选填"距离"（手动输入），用于月/年汇总

**路径 C（v1.1，不进 MVP）：App 内置 GPS 追踪**
- 跑步/骑行/徒步进入后台定位记录，CoreLocation + MapKit，实时配速、路径地图、暂停/继续
- 工作量预估 5–7 天，涉及后台定位、电池优化、路径持久化、权限引导，独立迭代

### 6.4 汇总图（日/月/年三维度）

| 维度 | 展示内容 |
|------|---------|
| 日 | 今日总消耗 kcal、运动时长、按类型饼图、已记录 workout 列表 |
| 月 | 月度总里程（跑步/骑行/走路分开栏）、月度运动小时、运动天数、streak 徽章、每日热力图 |
| 年 | 年度总里程、年度运动小时、年度总消耗、最长一次 workout、参与运动种类、"Apple 年末总结"式长截图可分享 |

### 6.5 热量方程式整合

首页热量公式由：
```
目标 = TDEE − 已摄入
```
改为：
```
今日可吃 = TDEE + 今日运动消耗 − 已摄入
```
运动消耗实时反馈到首页热量杯（同步手表 workout 结束后，首页数字自动 +X kcal，给用户正反馈）。

---

## 七、数据模型（SwiftData）

```swift
@Model FoodItem       // 食物库：名称、别名、每 100g 热量、宏量（蛋/脂/碳）、份量单位、份量大小、品牌、分类、来源（L1 seed / L2 user / L3 ai）
@Model MealEntry      // 一餐：日期、餐次、foodItemId、份量、热量、备注、来源（手动/AI/收藏）
@Model FastingSession // 一次断食：方案、开始、计划结束、实际结束、时长、状态、备注
@Model WeightEntry    // 一次称重：日期、体重 kg、体脂%、腰围 cm、来源（手动/HealthKit）
@Model WorkoutEntry   // 一次运动：类型、开始、时长、强度、消耗 kcal、距离 km（可空）、平均心率（可空）、来源（HealthKit / Manual）、HK UUID（去重用）
@Model WorkoutType    // 运动类型词典：ID、名称、图标、默认 MET、是否支持距离、分类（常规/传统养生）
@Model DietPlan       // 减肥计划：起止日期、TDEE、每日目标热量、断食方案、目标体重、活动量、状态
@Model UserProfile    // 用户档案：性别、生日、身高、初始体重、目标体重、偏好单位、禁用人群自查结果、免责声明同意时间
@Model AIConfig       // AI 配置：平台、baseURL、keychainRef、modelId、参数
```

核心算法：
- **BMR**（Mifflin-St Jeor）→ **TDEE**（活动系数 1.2–1.9）→ **每日目标** = TDEE − 建议缺口（默认 −500 kcal，可调 −250 / −500 / −750）
- **预计达成周期**：`(当前体重 − 目标体重) × 7700 kcal/kg ÷ 日缺口`
- **断食有效窗口**：≥ 13 小时算有效，用于 streak 计数

---

## 八、技术架构（沿用 matrixApps 体系）

```
LeanUp_App/
├── docs/
│   ├── LeanUp_PRD_v1.0.md           # 产品需求（本 Plan 扩写）
│   ├── LeanUp_详细设计.md            # 技术详设（模块/数据/算法/页面）
│   ├── LeanUp_UI设计与交互.md
│   └── LeanUp_避坑清单.md            # App Store 合规检查表 + 地雷词清单
├── ios_workspace/
│   ├── project.yml                   # xcodegen 2.41 (objectVersion≤56)
│   ├── LeanUp/
│   │   ├── App/                      # LeanUpApp, RootView, OnboardingFlow
│   │   ├── Features/
│   │   │   ├── Dashboard/
│   │   │   ├── Fasting/
│   │   │   ├── Meal/
│   │   │   ├── Weight/
│   │   │   ├── Workout/              # 运动记录 + 日/月/年汇总
│   │   │   ├── Plan/
│   │   │   └── Settings/
│   │   ├── Services/                 # HealthKitService, WorkoutSync, NotificationService, AIService(BYOK), FoodLibrary, SafetyGuard
│   │   ├── Models/                   # SwiftData @Model
│   │   ├── Widgets/
│   │   └── Resources/                # 食物库 seed json、运动 MET 词典、i18n、Assets
│   └── Assets.xcassets
└── tasks.yaml                        # matrixAppsCore 任务编排
```

- **平台基线**：iOS 17 / Swift 5.10 / SwiftUI / SwiftData / swift-tools-version 5.9
- **Shared 复用**：`MyAppBilling`（订阅 + 付费墙 + Quota）、`MyAppUI`（主题、设置页）、`MyAppCompliance`（隐私、ATT、免责声明）、`MyAppAnalytics`
- **广告**：复用 FireUp/SilenceCut 的 TopOn 统一广告管理，激励视频用于"解锁 AI 诊断周报预览 / 多一次计划重算"等非核心场景
- **HealthKit**（Apple 原生健康中枢）：
  - 读：体重（支持小米/华为/Withings 等体脂秤自动同步，免手输）、身高、性别、生日、步数、活动能量、**HKWorkout**（读取 Apple Watch / Strava / Nike / Keep 等任何 App 写入的运动记录，含类型/时长/距离/心率/消耗）
  - 写：LeanUp 里记录的体重可选写回系统，供 Apple Watch 和其他 App 读取
  - 用途：体重自动同步、运动 workout 自动入账、TDEE 根据实际活动量动态微调、新手引导少填字段
  - 权限分项请求，用户拒绝任何一项都能正常使用 App
- **SafetyGuard 服务**（合规中枢，独立一层）：
  - 每日目标热量下限强制校验（女 1200 / 男 1500）
  - BMI 安全阈值校验（目标 BMI < 18.5 拦截）
  - 禁用人群自查状态管理（未完成引导 → 某些功能禁用）
  - 医学免责声明同意时间戳记录
- **iCloud 同步（v1.0 启用）**：
  - SwiftData + CloudKit 原生集成，所有核心数据（饮食/体重/断食/运动/计划）多设备同步
  - 端到端加密，Apple 原生保障，无需自建服务器
  - **敏感字段不同步**：`AIConfig` 的 Keychain reference 强制本地存储，API Key 不走云
  - 首启引导可选登录 iCloud，拒绝也能正常使用（纯本地模式）
  - 多设备冲突策略：Last-Writer-Wins，饮食/体重等按 timestamp 合并
- **离线优先**：所有核心记录 100% 离线可用；AI 需网络（用户自选平台）

---

## 九、变现策略（免费 + 激励广告 + 订阅）

| 档位 | 包含 | 价格 |
|------|------|------|
| Free | 断食计时全量、每日饮食记录、体重记录、基础计划、横幅 + 开屏广告、激励视频解锁部分功能 | 免费 |
| Pro 月 | 去广告 + 多计划/多方案对比 + 高级报表 + AI 功能入口（仍需自带 Key）+ Widget 全量 + 数据导出 + 无限食物自建 | **$1.99 / 月** |
| Pro 年 | 同上 + 7 天免费试用 | **$12.99 / 年** |
| Lifetime（v1.1 再上） | 同上 + 永久解锁 | **$29.99 一次性** |

- 激励广告解锁点（非核心，不影响基础体验）：  
  - 查看本周 AI 诊断预览（BYOK 前提下）  
  - 重新生成减肥计划（免费版每周 1 次 + 看广告 +1 次）  
  - 解锁 30 天/90 天报表
- 广告平台：**TopOn 聚合**（复用 FireUp / SilenceCut 统一接入方案）
- 订阅技术：StoreKit 2，复用 Shared `MyAppBilling`，价格定位在 App Store 主流减肥工具的中低档（Simple $19.99/月 / Zero $9.99/月，我们 $1.99 做低价切入）

---

## 十、合规与隐私（海外 App Store 避坑专章）

> **背景警示**：2026 年 4 月 Cal AI（主流减脂 App）因绕过 IAP 被 Apple 下架。本年度减肥类审核明显收紧，本节全部要点必须在 MVP 第一版就到位，不留技术债。

### 10.1 硬性要求（10 条红线，踩了必拒）

| # | 审核条款 | 要求 | LeanUp 应对 |
|---|---------|------|------------|
| 1 | **1.4.1 Physical Harm** | 所有健康建议必须有权威医学引文 | 设置里加 `Scientific References` 页，列 Mifflin-St Jeor(1990)、WHO BMI、NIH 断食研究、Compendium of Physical Activities 等 5–8 条公开学术引用 |
| 2 | **1.4.1** | 禁止鼓励极端行为 | 每日目标热量 < 1200（女）/ 1500（男） kcal 强制弹警示 + 用户手动二次确认；BMI < 18.5 拒绝生成减脂计划 |
| 3 | **1.1.6** | 禁止疗效承诺 | 文案/截图/描述禁用 `cure / treat / guaranteed / lose X kg in Y days`，改用 `track / log / support` |
| 4 | **3.1.1 IAP** | 禁止任何第三方支付（Cal AI 因此下架） | 严格 StoreKit 2，复用 `MyAppBilling`，零 Stripe/微信/支付宝/自建网关 |
| 5 | **3.1.2** | 订阅透明度 | Paywall 显示 "Auto-renewable"、价格、周期、取消路径；首启不强推付费墙 |
| 6 | **5.1.3 Health** | HealthKit 数据禁用于广告/第三方分析 | HealthKit 数据流独立隔离，TopOn SDK 不读任何 HK 字段；隐私政策单独声明 |
| 7 | **5.1.1 Privacy** | App Privacy Label 如实填写 | 饮食、体重、运动、HealthKit 全部勾选；AI 通道标注"Third-party User-Configured" |
| 8 | **1.3 Kids** | 未成年人保护 | Age Rating 设 17+；首启年龄门拦截 < 18 |
| 9 | **2.3 Accurate Metadata** | 截图/描述不能夸大 | 免费版用户看到的 UI 和截图一致，不把 Pro 功能放免费位置 |
| 10 | **1.1.4** | 禁止不当内容 | 不用前后对比图、不用裸露/近裸身材图；营销用数据曲线和功能截图 |

### 10.2 六类禁用人群（首启引导必问）

引导第三屏列出以下自查项，任何一项勾选都触发相应处理：

| 人群 | 处理 |
|------|------|
| 18 岁以下 | 硬拦截，退出 App |
| 孕妇 / 哺乳期 | 警示横幅 + 不建议使用 + 可选退出 |
| 进食障碍病史 | 警示横幅 + 建议先咨询医生 + 可选退出 |
| 糖尿病 / 正在服药 | 警示横幅 + 强烈建议医生指导 |
| BMI < 18.5 | 拦截减脂计划生成，只允许"体重管理"模式（不推减脂） |
| 近期手术 / 慢性病 | 警示 + 建议医生指导 |

用户同意时间戳写入 `UserProfile.disclaimerAgreedAt`，每次 App 大版本升级必须重新同意。

### 10.3 文案地雷词（提交前自动扫描）

**禁用**：`cure, treat, diagnose, guaranteed, lose X kg, medical, prescription, clinical, therapy, heal, doctor-approved, FDA`  
**替代**：`track, log, support, help, monitor, record, tool, assist`  
**执行**：提交前用 `rg` 脚本扫一遍 App Store 描述、截图文案、应用内所有字符串资源。

### 10.4 隐私政策必含条款（面向海外 + GDPR + CCPA）

1. **我们不提供 AI 服务**：所有 AI 功能由用户自选的第三方模型提供商提供，对话数据发送至用户配置的平台；本 App 不存储、不转发、不分析 AI 对话内容
2. **HealthKit 数据本地使用**：不上传、不共享、不用于广告定向；iCloud 同步走 Apple 原生端到端加密
3. **广告 SDK 声明**：TopOn 聚合，海外主力网络（AdMob / Meta Audience Network / AppLovin / Unity Ads），用户可在设置关闭个性化广告
4. **ATT 框架**：按 Apple 要求请求 IDFA 权限，拒绝也能正常使用
5. **GDPR（欧盟用户）**：明确数据主体权利（访问/更正/删除/可携带），Cookie/追踪器同意管理
6. **CCPA（加州用户）**：Do Not Sell My Personal Information 开关
7. **数据导出/删除**：提供一键导出（JSON/CSV）和一键清除全部本地+iCloud 数据

### 10.5 实施检查表（提交前逐条勾选）

见独立产出文档 **`Projects/LeanUp_App/docs/LeanUp_避坑清单.md`**，包含：
- 10 条硬性红线的 QA 用例
- 6 类禁用人群的引导流程截图
- 地雷词自动扫描脚本
- App Privacy Label 填写清单
- 首次提交可能被问到的 Review Note 模板
- GDPR + CCPA 合规要点清单
- Apple 审核被拒时的申诉话术模板

---

## 十一、产出物（本次 Plan 批准后会生成）

1. `Projects/LeanUp_App/docs/LeanUp_PRD_v1.0.md` —— 完整 PRD（按 FireUp 风格扩写本 Plan）
2. `Projects/LeanUp_App/docs/LeanUp_详细设计.md` —— 技术详设（数据模型、TDEE/MET 算法、BYOK AI 服务层、HealthKit 集成、SafetyGuard、食物库 seed 构建脚本、Widget）
3. `Projects/LeanUp_App/docs/LeanUp_UI设计与交互.md` —— 页面 wireframe（Dashboard / 断食环 / 记餐流 / 体重曲线 / 运动记录 / 日月年汇总 / 计划问卷 / 首启引导三屏 / 设置 / 付费墙）
4. **`Projects/LeanUp_App/docs/LeanUp_避坑清单.md`** —— App Store 合规检查表（10 条硬性红线 + 6 类禁用人群流程 + 地雷词清单 + QA 用例 + Review Note 模板）
5. `Projects/LeanUp_App/tasks.yaml` —— matrixAppsCore 任务编排（MVP 分 6 个 sprint）
6. 暂不 scaffold Xcode 工程；scaffold 作为后续独立任务，确认 PRD 后再做

---

## 十二、全部决策已确认（Plan 锁定）

| 项 | 决策 |
|----|------|
| 产品名 | **LeanUp** — Fasting · Calories · Weight |
| 产品形态 | 综合型 MVP（轻断食 + 热量 + 体重 + 运动记录） |
| 目标市场 | **纯海外**，首发美区，英文主语言 + 简体中文次 |
| 变现 | 免费 + 激励广告（TopOn 海外网络）+ 订阅去广告 |
| 订阅价格 | $1.99 / 月、$12.99 / 年（7 天试用），v1.1 加 $29.99 终身 |
| AI | BYOK（用户自带 URL / Key），零内置模型 |
| HealthKit | 接入（体重 + Workout 双通道），权限全部可选 |
| 食物库 | L1 USDA FoodData 精选 1000–1500 条 + 美英快餐连锁 + 海外华人高频中式；L2 用户自建；L3 AI 解析兜底 |
| 运动模块 | 18 种（含八段锦/太极/气功/五禽戏/广场舞/武术）+ HealthKit 自动 + 手动快记 + 日月年汇总；**不做**内置 GPS（v1.1）、动作库、训练、视频、社群、私教 |
| Widget & Live Activity | **v1.0 做**：主屏 Widget 三档 + 锁屏 Widget + 断食 Live Activity（动态岛/锁屏实时倒计时）+ 互动 Widget |
| iCloud 同步 | **v1.0 做**：SwiftData + CloudKit 原生，端到端加密；AI Key 强制本地不同步 |
| 合规 | 对标 Apple 2026 审核新政 + GDPR + CCPA：10 条硬性红线 + 6 类禁用人群 + 地雷词清单 + SafetyGuard 服务层 |
| 产出物路径 | 全部在 `Projects/LeanUp_App/docs/` 下：PRD / 详细设计 / UI / 避坑清单 / tasks.yaml |

**本次无待确认项。** 下一步：切换 agent 模式生成 5 份文档。
