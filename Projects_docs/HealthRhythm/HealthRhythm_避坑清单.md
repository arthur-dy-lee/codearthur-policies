# HealthRhythm — App Store 避坑清单

**版本：** v1.0
**日期：** 2026 年 4 月 16 日
**目标市场：** 海外（首发美区）
**对标标准：** Apple App Store Review Guidelines（2026 年 2 月版）+ GDPR + CCPA
**文档性质：** 合规红线手册 · 提交前必读 · QA 必核

---

## 目录

1. [背景与风险提示](#1-背景与风险提示)
2. [10 条硬性红线](#2-10-条硬性红线)
3. [6 类禁用人群引导流程](#3-6-类禁用人群引导流程)
4. [文案地雷词清单](#4-文案地雷词清单)
5. [App Privacy Label 填写](#5-app-privacy-label-填写)
6. [隐私政策必含条款](#6-隐私政策必含条款)
7. [订阅 & IAP 合规](#7-订阅--iap-合规)
8. [广告 & ATT 合规](#8-广告--att-合规)
9. [HealthKit 合规要点](#9-healthkit-合规要点)
10. [BYOK AI 合规特别说明](#10-byok-ai-合规特别说明)
11. [iCloud / CloudKit 合规](#11-icloud--cloudkit-合规)
12. [提交前 Review Note 模板](#12-提交前-review-note-模板)
13. [被拒申诉话术模板](#13-被拒申诉话术模板)
14. [提交前检查表（一页总览）](#14-提交前检查表一页总览)
15. [自动化扫描脚本](#15-自动化扫描脚本)
16. [附录 A：科学引文](#附录-a科学引文)
17. [附录 B：GDPR & CCPA 合规细则](#附录-bgdpr--ccpa-合规细则)

---

## 1. 背景与风险提示

### 1.1 为什么要有这份清单

- **2026 年 Apple 审核收紧**：自 2026 年初起，健康类 App（尤其减肥、断食、热量追踪）审核标准明显提高，引文要求、禁用人群警示、IAP 纯净度均被严格核查
- **Cal AI 下架事件**（2026 年 4 月）：知名卡路里追踪 App Cal AI 因绕过 IAP 使用 Stripe 支付通道被 Apple 从 App Store 下架，是本年度最大合规事件，所有减肥类 App 都在高压线上
- **GDPR / CCPA 覆盖**：纯海外上架，美加澳、欧盟、英国都有不同隐私法规

### 1.2 合规优先级

| 级别 | 含义 | 处理 |
|------|------|------|
| P0 | 踩到必拒（或下架） | 必须在 MVP 第一版就到位，不留技术债 |
| P1 | 被问询风险高 | 必须准备 Review Note 说明 |
| P2 | 加分项 | 建议做，降低审核反复概率 |

### 1.3 本清单适用范围

- MVP v1.0 提交前逐项核查
- 每次大版本升级（功能涉及健康/支付/隐私）重新过一遍
- 每半年 Apple 审核指南更新后同步

---

## 2. 10 条硬性红线

### 红线 1：健康建议必须有科学引文（P0）

**审核条款**：`Guideline 1.4.1 — Safety · Physical Harm`

**要求**：  
所有"每日建议热量"、"断食方案推荐"、"减脂速度选项"、"BMR/TDEE 计算"等带科学性质的内容，必须在 App 内可直达的页面列出权威引文来源。

**HealthRhythm 应对**：
- 在 `Settings → About → Scientific References` 放一个独立页面，列 5–8 条公开学术引文（详见附录 A）
- TDEE 计算说明旁加小字 "Based on Mifflin-St Jeor equation (1990)"
- 断食 zones 说明旁加 "Informational only — not medical advice"
- 所有触发计算的结果页（如"你的每日目标是 1,820 kcal"）下方都有可点击链接直达 References 页

**QA 用例**：
- [ ] Settings 里能找到 Scientific References 页
- [ ] 页面列出 ≥ 5 条引文，每条含标题 + 作者 + 年份 + 来源
- [ ] 所有 TDEE / BMI / 断食相关文案中的数值断言可追溯到某条引文

**Review Note 话术**（如被问）：  
> "All health calculations and recommendations in HealthRhythm are based on peer-reviewed research, with sources listed in Settings → Scientific References. The app does not provide medical advice and repeatedly reminds users to consult healthcare providers."

---

### 红线 2：严禁鼓励极端行为（P0）

**审核条款**：`Guideline 1.4.1`

**要求**：  
App 不能引导或允许用户设置对健康有明确危害的参数：
- 每日热量目标过低
- BMI 过低者继续减脂
- 断食时长过长
- 未成年人使用

**HealthRhythm 硬编码**：

| 场景 | 阈值 | 处理 |
|------|------|------|
| 女性每日热量目标 | < 1,200 kcal | 弹琥珀警告 + 用户二次确认（ack）才能保存；事件记 `risk_ack_low_calorie` |
| 男性每日热量目标 | < 1,500 kcal | 同上 |
| 目标体重使 BMI | < 18.5 | **硬拦截**，不允许生成减脂计划，切换"体重维持"模式 |
| 当前 BMI | < 18.5 | Onboarding 屏 5 显琥珀警告，禁用激进减速档位 |
| 单次断食 | > 24 小时 | 弹窗 "Extended fasting may require medical supervision"，需 ack |
| 连续断食方案 | OMAD / 20:4 + 健康自查勾选 eating disorder | 方案隐藏，选择器显 "🔒 Not available for your health profile" |
| 用户年龄 | < 18 | **硬拦截**，退出 App |
| 孕妇/哺乳期 | 勾选 | 首页顶部常驻琥珀横幅 "Please follow guidance from your healthcare provider" |

**QA 用例**：
- [ ] 在 Onboarding 设每日目标 1,000 kcal 时弹警告
- [ ] 输入 DOB 使年龄 < 18 时退出
- [ ] 设目标体重 55 kg（对应 BMI 17）时拦截
- [ ] 勾选 pregnant 后 OMAD 方案不可见

---

### 红线 3：禁止疗效承诺（P0）

**审核条款**：`Guideline 1.1.6 — Safety · Objectionable Content`

**要求**：  
文案、截图、App Store 描述、截图文字层、推送通知不得出现"保证减重 X kg"、"治愈肥胖"、"替代医生"等暗示或承诺。

**禁用词汇完整清单**见第 4 节。

**截图与营销素材规范**：
- ❌ "Lose 10 lbs in 7 days"
- ❌ "Clinically proven to..."
- ❌ Before/after 身材对比
- ❌ 医生白大褂形象（暗示医疗背书）
- ✅ "Track your fasting"
- ✅ "Log meals in seconds"
- ✅ "Stay on your plan"

**HealthRhythm 应对**：
- App Store 描述统一走 `Review → Marketing Copy` 合规审核流程
- 所有文案资源（`.strings` / `.xcstrings`）提交前运行本文档第 15 节的扫描脚本
- Push 推送文案 QA 专项检查

---

### 红线 4：IAP 绝对纯净（P0，本年度最高风险）

**审核条款**：`Guideline 3.1.1 — Business · Payments · In-App Purchase`

**要求**：  
数字内容/订阅/解锁功能的**所有**支付必须走 StoreKit，绝对禁止任何第三方支付通道（Stripe / PayPal / Square / 微信 / 支付宝 / 自建支付网关）。

**Cal AI 下架案例教训**：  
Cal AI 在 App 内用 Stripe 渲染了一个看起来像 iOS 系统付款的 sheet 收费，虽然技术上没用 IAP，但 Apple 判定为"误导性支付界面 + 绕过抽成"，直接从 App Store 下架。

**HealthRhythm 应对**：
- **只用** StoreKit 2 API，通过 Shared `MyAppIAP` 包接入
- **禁止集成** 任何第三方支付 SDK（Stripe / Adyen / Checkout.com 等）
- Paywall UI 上的所有价格标签必须来自 StoreKit `Product.displayPrice`，不能硬编码
- "Restore Purchase" 必须是 `AppStore.sync()` 调用，不是自建会员系统
- 网络请求 QA 时抓包，确认订阅流程内**零**外部支付域名请求

**QA 用例**：
- [ ] 用 Proxyman 抓包付费流程，仅见 Apple 域名（`*.itunes.apple.com` / `*.apple.com`）
- [ ] 代码搜索：`rg "stripe\|paypal\|square\|checkout" ios_workspace/` 结果为空
- [ ] Paywall 价格在测试环境正确显示沙盒价格

---

### 红线 5：订阅透明度（P0）

**审核条款**：`Guideline 3.1.2 — Subscriptions`

**Paywall 必含 UI 元素**：
1. ✅ 明确标注 `Auto-renewable subscription`
2. ✅ 每档价格 + 计费周期（例：`$12.99 / year`）
3. ✅ 免费试用条款（例：`7-day free trial, then $12.99/year`）
4. ✅ 取消说明（例：`Cancel anytime in Settings`）
5. ✅ "Restore Purchase" 按钮
6. ✅ 可达的 `Terms of Service` + `Privacy Policy` 链接
7. ✅ 明显的关闭按钮（`X`），不能强制购买才能退出

**HealthRhythm Paywall 底部固定文字模板**：
```
Auto-renewable subscription. Cancel anytime in Settings.
Subscription automatically renews unless canceled at least 
24 hours before the end of the current period.

Terms · Privacy · Restore Purchase
```

**禁止**：
- ❌ 首启立即弹付费墙（应让用户先体验核心功能）
- ❌ 隐藏关闭按钮或使其难以发现
- ❌ 用"Free"字样误导（必须说清"7 days free, then paid"）
- ❌ 年费选项默认勾选但"save X%"文字夸大

**QA 用例**：
- [ ] 付费墙关闭 X 按钮位于左上角可达位置，44×44 点击区域
- [ ] 所有必含文字都在可视区域内（无需滚动）
- [ ] Restore Purchase 在沙盒测试环境可用

---

### 红线 6：HealthKit 数据禁用于广告（P0）

**审核条款**：`Guideline 5.1.3 — Data Collection and Storage · Health`

**要求**：  
从 HealthKit 读取的任何数据（体重、身高、心率、活动能量、Workout 等）**不得**用于：
- 广告定向
- 第三方分析（Firebase / Mixpanel 等）
- 与第三方共享（除非用户明确同意具体接收方）

**HealthRhythm 应对**：
- `HealthKitService` 作为独立服务层，数据只进 SwiftData，不进任何分析 SDK
- TopOn 广告 SDK 不暴露 HealthKit 字段
- `MyAppAnalytics` 事件定义中所有健康相关属性手动加注释 `// Do NOT send to analytics`，代码 review 时强检查
- Privacy Policy 第 "Health data" 章节明确声明

**代码层硬约束**：
```swift
// HealthKitService.swift
struct HealthKitService {
    // MARK: - Health data usage policy
    // Per Apple Guideline 5.1.3, data from HealthKit MUST NOT be:
    //   - Sent to advertising SDKs (TopOn, AdMob, Meta, etc.)
    //   - Sent to third-party analytics (Firebase, Mixpanel, etc.)
    //   - Shared with any third party without explicit user consent
    // All HealthKit data is local-only or synced via Apple's own CloudKit.
}
```

**QA 用例**：
- [ ] 抓包确认广告 SDK 请求中无体重、身高、workout 字段
- [ ] `MyAppAnalytics` 事件清单人工 review，无 HealthKit 来源字段
- [ ] Privacy Policy 健康数据章节齐全

---

### 红线 7：App Privacy Label 如实填写（P0）

**审核条款**：`Guideline 5.1.1 — Data Collection`

**提交 App Store Connect 时必填项**：

**A. Data Used to Track You**（跨 App 追踪）  
- `Identifiers — Device ID`（仅当用户同意 ATT 后收集 IDFA）
- `Usage Data`（广告 SDK 曝光/点击计数，如通过 ATT）

**B. Data Linked to You**（与身份关联）  
- `Health & Fitness — Health`（HealthKit 读取的数据）
- `Health & Fitness — Fitness`（运动记录）
- `User Content — Customer Support`（用户反馈邮件）
- `Purchases — Purchase History`（订阅记录）

**C. Data Not Linked to You**（匿名）  
- `Diagnostics — Crash Data`（崩溃日志）
- `Diagnostics — Performance Data`（性能埋点）
- `Usage Data — Product Interaction`（匿名功能使用统计，用户可在设置关闭）

**D. AI 相关特别声明**  
- BYOK AI 通道勾选 `Third-Party SDKs: User-Configured AI Providers`，明确 HealthRhythm 不收集 AI 对话内容

**QA 用例**：
- [ ] App Store Connect 里的 Privacy Label 每项都能在代码中找到对应数据流
- [ ] 代码中没有 Privacy Label 未声明的额外数据收集

---

### 红线 8：17+ 年龄分级（P0）

**审核条款**：`Guideline 1.3 — Kids Category`

**要求**：  
减肥、断食、热量追踪类 App 不适合未成年人，必须设 Age Rating 17+。

**HealthRhythm 应对**：
- App Store Connect → Age Rating 选 17+
- 分级原因勾选："Frequent/Intense Medical/Treatment Information"
- Onboarding 屏 3 硬拦截 < 18 岁用户
- 在 Settings → About → Age Rating 说明页重申"Designed for adults"

**QA 用例**：
- [ ] App Store Connect Age Rating = 17+
- [ ] 输入 DOB 使年龄 17 岁时拦截生效
- [ ] 描述、截图、预览视频不含儿童形象

---

### 红线 9：截图与描述不夸大（P0）

**审核条款**：`Guideline 2.3.3 — Accurate Metadata`

**要求**：
- 截图展示的功能必须在 App 中实际可用
- 免费用户能看到的 UI 和截图一致（不能把 Pro 专属截图放免费位置）
- 截图数据必须真实（不用"假截图"展示不存在的功能）
- 描述文案不能过度营销

**HealthRhythm 应对**：
- 截图策略：
  - 截图 1：Dashboard 真实状态（数据用合理模拟，无夸张）
  - 截图 2：断食 Live Activity（真机截屏）
  - 截图 3：饮食记录列表
  - 截图 4：体重趋势图（展示 3 个月真实节奏，非"一个月瘦 10kg"）
  - 截图 5：运动年度总结
  - 截图 6：AI BYOK 配置（强调"Bring Your Own Key"）
- 截图上的文字层统一走合规 review（见第 4 节地雷词）
- v1.0 不在截图展示未上线功能（如内置 GPS 在 v1.0 不展示）

---

### 红线 10：禁止不当视觉内容（P0）

**审核条款**：`Guideline 1.1.4 — Objectionable Content`

**禁用**：
- ❌ 裸露或近裸身材图
- ❌ 性暗示视觉元素
- ❌ Before/after 对比（即使不裸露也不做）
- ❌ "理想身材"形象图（可能被判定为身材焦虑诱导）

**允许**：
- ✅ 抽象插画（植物、圆环、数字）
- ✅ 食物图标
- ✅ 数据可视化（曲线、饼图）
- ✅ 中性头像/卡通形象

---

## 3. 6 类禁用人群引导流程

### 3.1 流程总览

Onboarding 屏 4（健康自查）让用户勾选下列任何项后，触发相应处理。所有处理存入 `UserProfile.healthFlags: Set<HealthFlag>`。

### 3.2 各类人群处理

| # | 自查项（英文）| 处理级别 | 具体处理 |
|---|-------------|--------|---------|
| 1 | Under 18 years old | 硬拦截 | Onboarding 屏 3 DOB 计算后直接退出；显示"HealthRhythm isn't designed for people under 18" |
| 2 | Pregnant or breastfeeding | 软警告 | 可继续使用，但：① Home 顶部常驻琥珀横幅"Please follow guidance from your healthcare provider"；② 禁用 OMAD / 20:4 方案；③ 每日目标热量建议值 +300 kcal（孕哺期需求提高）|
| 3 | History of eating disorder | 软警告 + 功能屏蔽 | 可继续使用，但：① 禁用所有 < 14 小时断食以外方案；② 禁止激进减脂速度档位；③ 首页不显示 "calorie deficit" 文案，改显 "daily target"；④ 每周弹一次关怀提示 |
| 4 | Diabetes or taking medication | 软警告 | 常驻横幅建议医生监督；弹窗 "Please work with your healthcare provider"；不禁用功能 |
| 5 | Recent surgery (< 3 months) | 软警告 | 建议暂停使用 App 直到康复，提供"Pause for now"按钮 |
| 6 | BMI < 18.5 (计算得出) | 部分功能拦截 | 自动切换为"Weight maintenance"模式（不生成减脂计划）；弹窗"HealthRhythm is designed for weight loss or maintenance, not further reduction at your current weight" |

### 3.3 文案模板

**软警告横幅（Home 顶部）：**
```
💛 We care about your safety
Because you indicated [condition], please work 
with your healthcare provider while using HealthRhythm.
[Dismiss]    [Edit health profile]
```

**BMI 过低弹窗：**
```
Title: We care about you
Body: Based on your height (175 cm) and weight 
(55 kg), your BMI is 17.9, below the healthy range.

HealthRhythm is designed for weight loss or maintenance,
not further reduction at your current weight. We'll 
switch you to weight maintenance mode.

You can always speak with a healthcare provider 
for personalized guidance.

Primary button: OK, got it
Secondary button: Learn more about BMI
```

**未成年拦截：**
```
Title: Thanks for your interest
Body: HealthRhythm isn't designed for people under 18.
Nutrition and fasting needs are different while 
you're still growing.

We hope to see you when you're ready.

Primary button: Exit
```

### 3.4 记录与追溯

- 用户同意/ack 操作写入 `SafetyAckLog`（本地 SwiftData 表）
- 字段：`timestamp`, `ackType`, `userAge`, `bmi`, `targetCalorie`
- 用户每次触发 SafetyGuard 时都记录，便于审计

---

## 4. 文案地雷词清单

### 4.1 绝对禁用词（出现即拒风险）

| 英文禁用 | 原因 | 替代 |
|---------|------|------|
| cure | 疗效承诺 | — |
| treat | 医疗暗示 | — |
| diagnose | 医疗暗示 | — |
| guaranteed | 承诺结果 | supported by evidence |
| medical | 医疗暗示（除非引用"not medical advice"）| health |
| prescription | 药物暗示 | — |
| clinical | 医疗暗示 | evidence-based |
| therapy / therapeutic | 医疗暗示 | practice / routine |
| heal | 医疗暗示 | support |
| doctor-approved | 伪背书 | — |
| FDA approved | 监管暗示 | — |
| clinically proven | 伪临床背书 | — |
| lose X kg in Y days | 结果承诺 | track your progress |
| burn fat fast | 夸大 | track your workouts |
| melt pounds | 夸大 | — |
| miracle | 夸大 | — |
| detox | 伪科学 | — |
| cleanse | 伪科学 | — |

### 4.2 允许词（中性工具语）

| 允许 | 含义 |
|------|------|
| track | 记录 |
| log | 记录 |
| monitor | 观察 |
| support | 支持 |
| help | 帮助 |
| record | 记录 |
| tool | 工具 |
| assist | 辅助 |
| guide | 引导（非医疗意义的）|
| plan | 计划 |
| routine | 日常 |

### 4.3 灰色区（依语境使用，必须加免责）

| 灰色词 | 使用条件 |
|-------|---------|
| health | 可用，但周围需出现 "tool" / "track" 这类中性词 |
| fasting | 可用，周围需有 "informational only" 提示 |
| weight loss | 可用，但不能搭配 "guaranteed" / "fast" 修饰 |
| calorie deficit | 可用，需有引文支持 |
| metabolism | 可用，不能承诺"boost" |

### 4.4 中文地雷词（中文版本用户界面）

| 禁用 | 原因 | 替代 |
|------|------|------|
| 治愈 | 疗效承诺 | — |
| 治疗 | 医疗暗示 | — |
| 诊断 | 医疗暗示 | — |
| 保证减重 | 结果承诺 | 记录进度 |
| 医学/医疗 | 医疗暗示 | 健康 |
| 临床验证 | 伪背书 | 基于研究 |
| 排毒 | 伪科学 | — |
| 燃脂神器 | 夸大 | — |

### 4.5 文案审核触发点

提交前必过下列内容的地雷词扫描：
- App Store 描述（长版 + 短版）
- App Store 截图文字层
- App Store 关键字（Keywords）
- 应用内 `.strings` / `.xcstrings` 所有键值
- 推送通知模板
- 客服自动回复
- Privacy Policy / Terms of Service / Medical Disclaimer
- 官网（如果有）首页文字
- 社媒广告素材

---

## 5. App Privacy Label 填写

### 5.1 App Store Connect 操作步骤

1. App Store Connect → My Apps → HealthRhythm → App Privacy
2. Data Types 分类选择
3. 每类下选"Linked to You"/"Not Linked"/"Used for Tracking"
4. 勾选对应 Purpose（App Functionality / Analytics / Developer's Advertising / Third-Party Advertising / Product Personalization）

### 5.2 HealthRhythm 完整填写清单

#### Data Linked to the User

**Health & Fitness**
- Health Data
  - Purposes: App Functionality
  - Linked: Yes
  - Tracking: No
- Fitness Data
  - Purposes: App Functionality
  - Linked: Yes
  - Tracking: No

**User Content**
- Other User Content（饮食记录、体重记录、备注）
  - Purposes: App Functionality
  - Linked: Yes
  - Tracking: No

**Purchases**
- Purchase History
  - Purposes: App Functionality
  - Linked: Yes
  - Tracking: No

**Identifiers**
- User ID（仅 iCloud 用户，用于设备间同步）
  - Purposes: App Functionality
  - Linked: Yes
  - Tracking: No

**Contact Info（仅当用户使用"Contact Support"时）**
- Email Address
  - Purposes: App Functionality
  - Linked: Yes
  - Tracking: No

#### Data Not Linked to the User

**Diagnostics**
- Crash Data
  - Purposes: App Functionality
- Performance Data
  - Purposes: App Functionality

**Usage Data**
- Product Interaction（匿名功能使用计数）
  - Purposes: Analytics

#### Data Used to Track You（需 ATT 同意）

**Identifiers**
- Device ID (IDFA)
  - Purposes: Third-Party Advertising
  - Tracking: Yes（仅 ATT 同意后）

**Usage Data**
- Advertising Data
  - Purposes: Third-Party Advertising
  - Tracking: Yes（仅 ATT 同意后）

### 5.3 Third-Party SDKs 声明

在 App Privacy → Third-Party SDKs 列出：
- **TopOn** — Third-Party Advertising（仅 ATT 同意后）
- **AdMob** — Third-Party Advertising（via TopOn）
- **Meta Audience Network** — Third-Party Advertising（via TopOn）
- **AppLovin** — Third-Party Advertising（via TopOn）
- **Unity Ads** — Third-Party Advertising（via TopOn）

### 5.4 AI 通道特别声明

App Privacy Policy 内专章说明：
> **AI Services (User-Configured)**: HealthRhythm does not provide AI. Users optionally configure their own API key from OpenAI, Anthropic, Google, or any OpenAI-compatible provider. When users invoke AI features, their input is sent directly from their device to the configured endpoint. HealthRhythm does not store, log, or transmit AI conversation content to our servers. API keys are stored locally in the device Keychain.

---

## 6. 隐私政策必含条款

### 6.1 Privacy Policy 完整结构（英文版）

1. **Introduction**
2. **Information We Collect**
   - 2.1 Health & Fitness Data (HealthKit)
   - 2.2 User Content (meals, weight, workouts, plans)
   - 2.3 Diagnostic Data (crash, performance)
   - 2.4 Advertising Data (with ATT consent)
   - 2.5 Purchase History
3. **How We Use Your Information**
4. **Data Storage & Security**
   - 4.1 Local Storage (SwiftData)
   - 4.2 iCloud Sync (if enabled)
   - 4.3 Keychain (API keys)
5. **Third-Party Services**
   - 5.1 Apple HealthKit
   - 5.2 iCloud / CloudKit
   - 5.3 AI Services (User-Configured BYOK)
   - 5.4 Advertising (TopOn and its partners)
   - 5.5 Payment Processing (Apple)
6. **Your Rights**
   - 6.1 Access
   - 6.2 Correction
   - 6.3 Deletion
   - 6.4 Portability
   - 6.5 Opt-out of Advertising
7. **GDPR (EU/UK Users)**
8. **CCPA (California Users)**
9. **Children's Privacy**（明确 not designed for under 18）
10. **Changes to This Policy**
11. **Contact Us**

### 6.2 关键段落模板

#### Section 2.1 – Health & Fitness Data

```
If you grant permission, HealthRhythm reads the following types of
data from Apple Health:
- Body Mass (weight)
- Height
- Sex
- Date of Birth
- Step Count
- Active Energy Burned
- Workout sessions (type, duration, distance, heart rate, energy)

HealthRhythm may also write the following back to Apple Health:
- Body Mass (when you log weight)
- Workout sessions (when you log workouts manually)

This data is stored locally on your device and, if you enable 
iCloud Sync, synced across your devices via Apple's end-to-end 
encrypted CloudKit. We do NOT transmit health data to our own 
servers, third parties, or advertising networks.
```

#### Section 5.3 – AI Services (BYOK)

```
HealthRhythm does not provide AI services. If you choose to use AI
features (meal description parsing, weekly diet analysis, etc.), 
you must configure your own API credentials from a third-party 
AI provider (OpenAI, Anthropic, Google Gemini, or any 
OpenAI-compatible service).

When you invoke an AI feature:
- Your input (e.g., the text you wrote) is sent DIRECTLY from 
  your device to the endpoint you configured
- HealthRhythm does not store, log, or route this content through
  our servers
- Your API key is stored in the iOS Keychain on your device 
  and is never transmitted to our servers
- Each AI provider has its own privacy policy; please review 
  theirs before use

You can remove your AI configuration at any time in Settings, 
which clears the API key from your device.
```

#### Section 7 – GDPR

```
If you are in the European Union, European Economic Area, or 
United Kingdom, you have the following rights under GDPR:

- Right of access: Request a copy of your data
- Right to rectification: Correct inaccurate data
- Right to erasure: Delete your data
- Right to data portability: Export your data in JSON/CSV
- Right to object: Opt out of advertising tracking
- Right to lodge a complaint: Contact your local Data 
  Protection Authority

HealthRhythm's legal basis for processing:
- Consent (for HealthKit, advertising tracking, AI usage)
- Contract (for subscription services)
- Legitimate interest (for analytics, crash reporting)

To exercise your rights, use Settings → Export Data or 
Delete All Data, or email privacy@[domain].com.
```

#### Section 8 – CCPA

```
If you are a California resident, you have the right to:

- Know what personal information we collect
- Know whether your information is sold or disclosed
- Say "no" to the sale of personal information
- Access the information we have collected
- Request deletion

HealthRhythm does NOT sell your personal information.

To exercise your rights, visit Settings or email 
privacy@[domain].com.

Do Not Sell My Personal Information: [link to opt-out form]
```

### 6.3 Terms of Service 关键条款

#### Medical Disclaimer（必须单独显著章节）

```
IMPORTANT: HealthRhythm is a tracking tool, not medical advice.

HealthRhythm is designed to help you log and visualize your food
intake, fasting windows, weight, and workouts. It is NOT 
intended to:
- Diagnose, treat, cure, or prevent any disease
- Replace professional medical advice, diagnosis, or treatment
- Be used by people under 18 years of age
- Be used by pregnant or breastfeeding individuals without 
  healthcare provider supervision
- Be used by people with a history of eating disorders 
  without healthcare provider supervision

Always seek the advice of a qualified healthcare provider 
before starting any diet, fasting, or exercise program. 
Never disregard professional medical advice or delay 
seeking it because of something you have read in HealthRhythm.

If you experience any adverse health effects, stop using 
HealthRhythm immediately and consult a healthcare provider.

By using HealthRhythm, you acknowledge these limitations and
agree to use the app at your own discretion.
```

---

## 7. 订阅 & IAP 合规

### 7.1 订阅产品配置（App Store Connect）

| Product ID | 类型 | 价格 | 试用 | 自动续费 |
|-----------|------|------|------|---------|
| `com.HealthRhythm.pro.monthly` | Auto-Renewable Subscription | $1.99 | — | 是 |
| `com.HealthRhythm.pro.yearly` | Auto-Renewable Subscription | $12.99 | 7 days | 是 |
| （v1.1）`com.HealthRhythm.pro.lifetime` | Non-Consumable | $29.99 | — | — |

同一个 Subscription Group（`HealthRhythm Pro`），月/年可互换升降级。

### 7.2 Paywall UI 强制元素检查

| 元素 | 位置 | 必要性 |
|------|------|-------|
| "Auto-renewable subscription" 文字 | 付费墙底部可视区域 | 必 |
| 价格 + 周期（来自 StoreKit） | 每档位标签 | 必 |
| "7-day free trial" 文字（年费档） | 年费档位标签 | 必（当有试用时） |
| "Cancel anytime in Settings" | 底部 | 必 |
| Restore Purchase 按钮 | 底部 | 必 |
| Terms of Service 链接 | 底部 | 必 |
| Privacy Policy 链接 | 底部 | 必 |
| 关闭按钮（X）| 左上或右上 | 必 |

### 7.3 测试清单

- [ ] 沙盒测试：订阅成功 → entitlements 正确同步
- [ ] 沙盒测试：取消订阅 → 到期后失去 Pro 权益
- [ ] 沙盒测试：升级月 → 年 按比例抵扣
- [ ] 沙盒测试：家庭共享（Family Sharing）正常
- [ ] Restore Purchase 在新设备登录后正确恢复

### 7.4 禁止行为

- ❌ 强制用户购买才能使用核心功能（必须有免费可用的基本版）
- ❌ 用"Free!" 字样误导（必须说清后续计费）
- ❌ 关闭按钮隐藏或难以发现
- ❌ 虚假紧迫感（"Only 2 hours left!" 等）
- ❌ 在付费墙中使用任何非 StoreKit 的付款 UI

---

## 8. 广告 & ATT 合规

### 8.1 ATT (App Tracking Transparency) 流程

1. App 首次启动（或合适时机）调用 `ATTrackingManager.requestTrackingAuthorization`
2. 在请求前通过自定义 pre-prompt 解释为什么需要追踪（提高同意率）
3. 记录用户同意/拒绝状态到本地
4. **不论用户是否同意，App 功能必须完全可用**
5. 未同意时，广告 SDK 关闭个性化追踪（non-personalized ads）

### 8.2 Pre-Prompt 文案（合规 + 转化双目标）

```
Title: Keep HealthRhythm free with personalized ads
Body: Allowing tracking helps us show you more relevant 
ads and keeps HealthRhythm free. Your health data is never
shared with advertisers.
Buttons:
- Allow (→ 触发 ATT 系统弹窗)
- Not now (→ 不触发系统弹窗，继续使用)
```

### 8.3 TopOn 配置要点

- 海外启用网络：AdMob / Meta Audience Network / AppLovin / Unity Ads / Pangle（海外版）
- **禁用**：国内穿山甲、优量汇、快手
- 配置激励广告 placementID 独立于横幅/开屏
- 广告频控：横幅不超过 2 次/分钟，插屏不超过 1 次/3 分钟

### 8.4 合规硬约束

- 广告 SDK 不得读 HealthKit 数据（代码审查）
- 广告内容不得涉及处方药、减肥药、医疗服务（TopOn 可在后台过滤类别）
- 儿童不适宜内容必须过滤

---

## 9. HealthKit 合规要点

### 9.1 权限申请文案（`Info.plist`）

```xml
<key>NSHealthShareUsageDescription</key>
<string>HealthRhythm reads your weight, height, and workout data to keep your tracking in sync with Apple Health. Your health data never leaves your device except through iCloud Sync (end-to-end encrypted by Apple).</string>

<key>NSHealthUpdateUsageDescription</key>
<string>HealthRhythm writes your weight and workouts to Apple Health so they're available in the Health app and other apps you use.</string>
```

### 9.2 数据类型分类申请

每项权限独立请求，用户可细粒度选择：
- Body Mass (read + write)
- Height (read)
- Biological Sex (read)
- Date of Birth (read)
- Step Count (read)
- Active Energy Burned (read)
- Workout (read + write)

### 9.3 必须能拒绝

- 用户拒绝任何一项或全部，App 必须继续可用（降级为手动录入）
- 设置里提供 "Open Health app" 深链让用户修改

### 9.4 数据流硬约束（重申红线 6）

```
┌─────────────┐       ┌──────────────┐       ┌──────────────┐
│ Apple Health│ ====> │ HealthKitSvc │ ====> │  SwiftData   │
└─────────────┘       └──────────────┘       └──────────────┘
                             │
                             ├─X─> Analytics SDK  (BANNED)
                             ├─X─> Ad SDK         (BANNED)
                             └─X─> Server         (BANNED)
```

---

## 10. BYOK AI 合规特别说明

### 10.1 为什么 BYOK 是合规利器

- **零生成式 AI 备案责任**：中国网信办生成式 AI 算法备案不适用（不涉及中国区）；Apple 审核时也无需做 AI 安全说明（因为我们不提供 AI）
- **零 AI 内容责任**：用户对接的平台（OpenAI / Anthropic 等）自己已经过合规审查
- **零 AI 滥用风险**：用户自担 API 费用，我们无侵权/越权风险

### 10.2 App Store 审核时的 Review Note

```
HealthRhythm does not provide any AI or LLM services. The AI
features (meal description parsing, diet analysis) are 
entirely optional and require users to configure their own 
API credentials from third-party providers (OpenAI, 
Anthropic, Google, or any OpenAI-compatible service).

When a user invokes an AI feature:
- The user's input is sent directly from their device to 
  their configured endpoint
- HealthRhythm does not host, relay, or proxy any AI traffic
- HealthRhythm does not store API keys on our servers; keys are
  stored in the device Keychain

This BYOK (Bring Your Own Key) architecture means:
- HealthRhythm is not responsible for AI-generated content
- User privacy is maintained between user and their chosen 
  AI provider
- Users pay their AI provider directly for usage
```

### 10.3 UI 文案严禁表述

- ❌ "HealthRhythm's AI can tell you what to eat"
- ❌ "Powered by AI"（在 App 描述中不出现）
- ❌ 任何暗示 HealthRhythm 自带 AI 的文案
- ✅ "Optional AI features (requires your own API key)"
- ✅ "Bring your own AI"
- ✅ "Configure your preferred AI provider"

### 10.4 API Key 安全要求

- **存储**：iOS Keychain，`kSecAttrAccessibleAfterFirstUnlock`
- **不同步**：Keychain 同步功能关闭（`kSecAttrSynchronizable = false`），Key 不进 iCloud
- **不日志**：崩溃日志、性能埋点严禁包含 Key
- **可清除**：Settings 提供"Clear AI credentials"按钮

---

## 11. iCloud / CloudKit 合规

### 11.1 为什么用 CloudKit 合规零成本

- Apple 原生端到端加密（E2EE 部分字段）
- 用户身份绑定 Apple ID，不收集任何注册信息
- Apple 已经过 GDPR / CCPA / HIPAA 合规审查
- 数据存储在用户自己的 iCloud 空间

### 11.2 CloudKit 配置

- 使用 `CKContainer.default()`（与 App Bundle ID 绑定）
- 使用 `CKContainer.privateCloudDatabase`（私人数据）
- SwiftData 原生集成：`ModelConfiguration(cloudKitDatabase: .automatic)`
- 所有字段必须符合 CloudKit schema 要求（默认值、可选性）

### 11.3 不同步的字段

- `AIConfig.apiKey` → 强制本地（Keychain）
- `UserProfile.healthFlags` → 同步（方便多设备一致体验）
- `SafetyAckLog` → 同步（审计用途）

### 11.4 冲突策略

- Last-Writer-Wins（按 `modifiedAt` 时间戳）
- 饮食记录按 `(date, meal, foodId, timestamp)` 去重
- 体重记录按 `(date, source)` 去重
- Workout 按 `(hkUUID)` 去重（HealthKit UUID 唯一）

### 11.5 用户体验

- Settings → iCloud Sync toggle 可关闭
- 关闭时本地数据保留，但不再上传
- 重新开启时提示可能冲突

---

## 12. 提交前 Review Note 模板

### 12.1 App Store Connect → App Review Information

**Notes to Reviewer**（完整模板）：

```
Hi App Review Team,

Thank you for reviewing HealthRhythm.

HealthRhythm is a privacy-first, offline-first weight management
tracker focused on fasting, calorie logging, weight monitoring, 
and workout recording. Below is relevant context for review:

---

## Account / Login
No account required. HealthRhythm is fully functional without
registration. iCloud Sync is optional and uses Apple's own 
infrastructure (no third-party servers).

Test account: Not applicable — no login required.

## Subscription
HealthRhythm offers optional auto-renewable subscriptions:
- Pro Monthly: $1.99/month
- Pro Yearly: $12.99/year (with 7-day free trial)

All subscriptions use StoreKit 2 exclusively. No third-party 
payment processors are used.

## AI Features (BYOK)
HealthRhythm does NOT provide AI. AI features are optional and
require users to configure their own API key from third-party 
providers (OpenAI, Anthropic, Google Gemini, or any 
OpenAI-compatible service). Their input is sent directly 
from the user's device to their configured endpoint.

Test AI: To test, reviewer may skip AI setup — all core 
features work without AI.

## HealthKit
HealthRhythm reads weight, height, sex, DOB, steps, active energy,
and workouts from Apple Health, and writes weight and workouts 
back. Users can grant each permission individually; all 
features are usable without any permission.

HealthKit data is stored locally (and optionally in the user's 
own iCloud via end-to-end encrypted CloudKit). It is NEVER 
transmitted to our servers, advertisers, or analytics.

## Age Rating
17+ (Frequent/Intense Medical/Treatment Information).
Users under 18 are blocked at onboarding.

## Scientific Basis
All health calculations are based on peer-reviewed research. 
See Settings → Scientific References for citations:
- Mifflin-St Jeor equation (1990) for BMR/TDEE
- WHO BMI classification
- NIH fasting research
- Compendium of Physical Activities (Ainsworth et al.) for 
  exercise MET values

## Medical Disclaimer
Prominently displayed at onboarding (mandatory acknowledgment) 
and accessible in Settings. App does not diagnose, treat, or 
cure any condition.

## Ads
Ads are served via TopOn aggregation (AdMob, Meta Audience 
Network, AppLovin, Unity Ads). ATT is implemented per 
Apple guidelines. HealthKit data is never shared with 
advertisers.

## GDPR / CCPA
Full compliance. Users can export and delete all data from 
Settings. See Privacy Policy for details.

## Contact
For any questions: [support email]
Privacy concerns: [privacy email]

Best regards,
HealthRhythm Team
```

### 12.2 Demo Account（如果需要）

- 无登录所以不需要 demo 账号
- 在 Notes 中明确写："No login required"

---

## 13. 被拒申诉话术模板

### 13.1 场景：被拒原因为"健康建议缺引文"

```
Dear App Review Team,

Thank you for your feedback regarding our health-related 
calculations.

HealthRhythm includes a dedicated "Scientific References" page
in Settings → About, which lists peer-reviewed sources 
for all health-related calculations:

- BMR/TDEE: Mifflin, M. D., et al. "A new predictive 
  equation for resting energy expenditure in healthy 
  individuals." Am J Clin Nutr, 1990.
- BMI: World Health Organization. "Global Database on Body 
  Mass Index." WHO, 2004.
- Exercise MET values: Ainsworth, B. E., et al. 
  "Compendium of Physical Activities." Med Sci Sports 
  Exerc, 2011.

We have updated our in-app messaging at [specific locations] 
to include direct links to this References page, ensuring 
users can verify sources at any time.

We would be happy to provide additional context or make 
further adjustments as needed.

Best regards,
HealthRhythm Team
```

### 13.2 场景：被拒原因为"疗效承诺"

```
Dear App Review Team,

Thank you for flagging the language concern. We have 
reviewed and updated the following:

Before: "[原词]"
After: "[替代词]"

Specific changes:
1. App Store description: [diff]
2. In-app copy at [location]: [diff]
3. Screenshot text at [location]: [diff]

HealthRhythm is a tracking tool, not medical advice, and we've
strengthened our Medical Disclaimer presentation at 
onboarding and in Settings.

A new binary has been submitted reflecting these changes.

Best regards,
HealthRhythm Team
```

### 13.3 场景：被拒原因为"IAP 相关"

```
Dear App Review Team,

Thank you for your review. We confirm that HealthRhythm uses
StoreKit 2 exclusively for all subscriptions and in-app 
purchases. No third-party payment processors are 
integrated.

Specifically:
- All subscription purchases go through Apple's StoreKit 2
  via the MyAppIAP framework
- Restore Purchase uses AppStore.sync()
- No network requests to Stripe, PayPal, or any payment 
  processor are made
- Paywall UI uses Product.displayPrice from StoreKit 
  (no hard-coded prices)

We have included network trace logs demonstrating only 
Apple domains are contacted during the purchase flow. 
Available upon request.

Best regards,
HealthRhythm Team
```

### 13.4 场景：被拒原因为"Age Rating / 未成年人保护"

```
Dear App Review Team,

Thank you for your feedback. We have implemented the 
following safeguards for underage users:

1. Age Rating set to 17+ in App Store Connect
2. Onboarding requires date of birth entry
3. Users under 18 are hard-blocked from proceeding past 
   onboarding; they see an explanatory screen and cannot 
   access app features
4. Medical Disclaimer explicitly states the app is not 
   designed for users under 18
5. Test: Try entering a DOB of [2010] — you'll see the 
   block screen

A new binary reflecting these safeguards has been 
submitted.

Best regards,
HealthRhythm Team
```

---

## 14. 提交前检查表（一页总览）

打印本页贴显示器前，每次提交前逐条勾选。

### ✅ 代码层

- [ ] Age Rating 设为 17+
- [ ] DOB < 18 硬拦截生效
- [ ] 每日目标热量 < 1200(F)/1500(M) 弹警告
- [ ] 目标 BMI < 18.5 拦截
- [ ] 6 类禁用人群引导齐全
- [ ] SafetyGuard 服务覆盖所有入口
- [ ] 免责声明同意时间戳写入 UserProfile
- [ ] Scientific References 页可达
- [ ] 所有健康计算可追溯引文
- [ ] HealthKit 数据零进广告/分析 SDK
- [ ] 抓包验证：付费流程只有 Apple 域名
- [ ] 代码搜索 "stripe\|paypal\|checkout" 结果为空
- [ ] ATT 权限申请前有 pre-prompt
- [ ] 用户拒绝 ATT / HealthKit 后 App 仍可用

### ✅ 文案层

- [ ] 运行地雷词扫描脚本，零匹配
- [ ] App Store 描述 review 通过
- [ ] App Store 关键字 review 通过
- [ ] 所有截图文字 review 通过
- [ ] 推送通知模板 review 通过
- [ ] 中英文版本都已 review

### ✅ 付费墙

- [ ] "Auto-renewable subscription" 文字可见
- [ ] "Cancel anytime" 文字可见
- [ ] "7-day free trial" 文字（年费档）可见
- [ ] Restore Purchase 按钮可点击
- [ ] Terms + Privacy 链接可点击
- [ ] 关闭按钮清晰可达
- [ ] 沙盒购买成功
- [ ] 沙盒取消后权益失效
- [ ] 新设备 Restore Purchase 可用

### ✅ App Store Connect

- [ ] Age Rating: 17+
- [ ] Primary Category: Health & Fitness
- [ ] Secondary Category: Food & Drink
- [ ] App Privacy 填写完整（每项数据类型有对应代码）
- [ ] Third-Party SDKs 列出 TopOn 及其网络
- [ ] Notes to Reviewer 填写（用第 12 节模板）
- [ ] Demo Video 录制（展示 onboarding、记餐、断食、订阅流程）

### ✅ 法律文档

- [ ] Privacy Policy 发布且 URL 填入 App Store Connect
- [ ] Terms of Service 发布
- [ ] Medical Disclaimer 单独页面可达
- [ ] GDPR + CCPA 条款齐全

### ✅ 隐私 & 数据

- [ ] Info.plist 所有 NSxxxUsageDescription 文案清晰
- [ ] HealthKit 权限分项申请
- [ ] 数据导出功能测试通过（JSON + CSV）
- [ ] 数据清除功能测试通过（本地 + iCloud）

### ✅ 广告

- [ ] TopOn 海外渠道启用，国内渠道禁用
- [ ] 广告频控符合设计
- [ ] 广告内容类别过滤（无处方药/减肥药/医疗）
- [ ] 广告 SDK 代码审查：无 HealthKit 数据暴露

---

## 15. 自动化扫描脚本

### 15.1 地雷词扫描（Python）

保存为 `scripts/check_compliance.py`，提交前运行。

```python
#!/usr/bin/env python3
"""HealthRhythm compliance word scanner."""
import re
import sys
from pathlib import Path

BANNED_EN = [
    r'\bcure\b', r'\btreat\b', r'\bdiagnose\b', r'\bguaranteed\b',
    r'\bprescription\b', r'\bclinical\b', r'\btherap(y|eutic)\b',
    r'\bheal\b', r'\bdoctor-approved\b', r'\bFDA approved\b',
    r'\bclinically proven\b', r'lose \d+ (kg|lbs|pounds) in \d+',
    r'\bburn fat fast\b', r'\bmelt pounds\b', r'\bmiracle\b',
    r'\bdetox\b', r'\bcleanse\b',
]

BANNED_ZH = [
    r'治愈', r'治疗', r'诊断', r'保证减重', r'医学', r'医疗',
    r'临床验证', r'排毒', r'燃脂神器', r'瘦身神器',
]

ALLOWED_CONTEXTS = [
    r'not medical advice',
    r'do not diagnose',
    r'consult.*medical',
    r'professional medical advice',
    r'非医疗建议',
    r'不能诊断',
]

def scan_file(path: Path) -> list:
    if not path.suffix in {'.md', '.swift', '.strings', '.xcstrings', '.txt', '.html', '.json'}:
        return []
    
    content = path.read_text(encoding='utf-8', errors='ignore')
    hits = []
    
    for pattern in BANNED_EN + BANNED_ZH:
        for match in re.finditer(pattern, content, re.IGNORECASE):
            line_num = content[:match.start()].count('\n') + 1
            line = content.split('\n')[line_num - 1]
            
            is_allowed = any(re.search(ctx, line, re.IGNORECASE) for ctx in ALLOWED_CONTEXTS)
            if is_allowed:
                continue
            
            hits.append((str(path), line_num, pattern, line.strip()))
    
    return hits

def main():
    root = Path(sys.argv[1] if len(sys.argv) > 1 else '.')
    all_hits = []
    
    for path in root.rglob('*'):
        if any(p in str(path) for p in ['.git', 'Pods', 'build', 'DerivedData', 'node_modules']):
            continue
        if path.is_file():
            all_hits.extend(scan_file(path))
    
    if not all_hits:
        print('✓ No banned words found')
        return 0
    
    print(f'✗ Found {len(all_hits)} potential issue(s):\n')
    for file, line_num, pattern, text in all_hits:
        print(f'  {file}:{line_num}')
        print(f'    Pattern: {pattern}')
        print(f'    Line:    {text}')
        print()
    
    return 1

if __name__ == '__main__':
    sys.exit(main())
```

用法：
```bash
cd Projects/HealthRhythm_App
python scripts/check_compliance.py ios_workspace/
python scripts/check_compliance.py docs/
```

### 15.2 支付通道扫描（ripgrep）

```bash
#!/bin/bash
# scripts/check_no_third_party_payment.sh

echo "Scanning for forbidden payment integrations..."
HITS=$(rg -i "stripe|paypal|square|checkout\.com|adyen|braintree|alipay|wechat.?pay" \
       ios_workspace/ \
       --type swift --type plist --type json 2>/dev/null)

if [ -n "$HITS" ]; then
    echo "✗ Found payment-related keywords (must use StoreKit only):"
    echo "$HITS"
    exit 1
else
    echo "✓ No forbidden payment SDK references found"
fi
```

### 15.3 HealthKit 数据泄露扫描

```bash
#!/bin/bash
# scripts/check_healthkit_leak.sh

echo "Scanning for HealthKit data being sent to analytics/ads..."

# Check if any analytics event contains HealthKit-related field names
SUSPICIOUS=$(rg -i "Analytics\.log|Ad.*track|logEvent" ios_workspace/ --type swift -A 5 | \
             rg -i "weight|bodyMass|healthKit|workout|heartRate|activeEnergy")

if [ -n "$SUSPICIOUS" ]; then
    echo "⚠ Potential HealthKit data leak (manual review required):"
    echo "$SUSPICIOUS"
    exit 1
else
    echo "✓ No obvious HealthKit data leaks"
fi
```

### 15.4 集成到 CI

在 `.github/workflows/compliance.yml` 或 fastlane action 中：
```yaml
- name: Compliance scan
  run: |
    python scripts/check_compliance.py ios_workspace/
    bash scripts/check_no_third_party_payment.sh
    bash scripts/check_healthkit_leak.sh
```

提交前必须全部通过。

---

## 附录 A：科学引文

用于 `Settings → About → Scientific References` 页面展示。

### A.1 BMR / TDEE

1. Mifflin, M. D., St Jeor, S. T., Hill, L. A., Scott, B. J., Daugherty, S. A., & Koh, Y. O. (1990). **A new predictive equation for resting energy expenditure in healthy individuals**. The American Journal of Clinical Nutrition, 51(2), 241–247. https://doi.org/10.1093/ajcn/51.2.241

2. Harris, J. A., & Benedict, F. G. (1918). **A biometric study of human basal metabolism**. PNAS, 4(12), 370–373. https://doi.org/10.1073/pnas.4.12.370

### A.2 BMI 分类

3. World Health Organization. (2004). **Appropriate body-mass index for Asian populations and its implications for policy and intervention strategies**. The Lancet, 363(9403), 157–163.

4. WHO Expert Consultation. (1995). **Physical Status: The Use and Interpretation of Anthropometry**. WHO Technical Report Series 854.

### A.3 Fasting

5. de Cabo, R., & Mattson, M. P. (2019). **Effects of intermittent fasting on health, aging, and disease**. New England Journal of Medicine, 381(26), 2541–2551.

6. Patterson, R. E., & Sears, D. D. (2017). **Metabolic effects of intermittent fasting**. Annual Review of Nutrition, 37, 371–393.

### A.4 Exercise MET

7. Ainsworth, B. E., Haskell, W. L., Herrmann, S. D., Meckes, N., Bassett, D. R., Tudor-Locke, C., … & Leon, A. S. (2011). **2011 Compendium of Physical Activities: a second update of codes and MET values**. Medicine & Science in Sports & Exercise, 43(8), 1575–1581.

### A.5 Weight Loss General

8. Hall, K. D., Sacks, G., Chandramohan, D., Chow, C. C., Wang, Y. C., Gortmaker, S. L., & Swinburn, B. A. (2011). **Quantification of the effect of energy imbalance on bodyweight**. The Lancet, 378(9793), 826–837.

### A.6 Traditional Chinese Exercise

9. Xiong, X., Wang, P., Li, X., Zhang, Y., & Li, S. (2015). **The effects of Tai Chi on heart rate variability in older Chinese individuals with depression**. International Journal of Environmental Research and Public Health, 12(2), 2162–2177.

10. Wayne, P. M., Walsh, J. N., Taylor-Piliae, R. E., Wells, R. E., Papp, K. V., Donovan, N. J., & Yeh, G. Y. (2014). **Effect of Tai Chi on cognitive performance in older adults: systematic review and meta-analysis**. Journal of the American Geriatrics Society, 62(1), 25–39.

---

## 附录 B：GDPR & CCPA 合规细则

### B.1 GDPR 数据主体权利实现

| 权利 | HealthRhythm 实现 |
|------|------------|
| Right of Access | Settings → Export Data（JSON/CSV 全量） |
| Right to Rectification | 所有记录均可在 App 内编辑 |
| Right to Erasure | Settings → Delete All Data（本地 + iCloud） |
| Right to Data Portability | Export 支持标准 JSON/CSV |
| Right to Restrict Processing | 关闭各类同步/分析 toggle |
| Right to Object | 关闭个性化广告 toggle |
| Right to Lodge Complaint | Privacy Policy 列出各国 DPA 链接 |

### B.2 GDPR 法律依据

- **Consent** — HealthKit、广告追踪、AI 调用
- **Contract** — 订阅服务、核心记录功能
- **Legitimate Interest** — 崩溃报告、性能埋点（匿名）

### B.3 CCPA 必做项

- [ ] Privacy Policy 加 "Do Not Sell My Personal Information" 章节
- [ ] Settings 加 "Do Not Sell My Personal Information" toggle（即使我们不卖，也要有入口）
- [ ] 12 个月内响应数据请求
- [ ] 不对行使权利的用户歧视定价

### B.4 Cookie / Tracking 同意管理

- iOS App 不涉及 Cookie
- ATT 框架满足 Apple 对 cross-app tracking 的同意要求
- 欧盟用户额外看到一次"我们使用匿名分析"横幅（可拒绝）

---

## 附录 C：文档版本与更新记录

| 版本 | 日期 | 变更 |
|------|------|------|
| v1.0 | 2026-04-16 | 初版，对齐 Apple 2026-02 审核指南 + Cal AI 下架案分析 |

---

**文档结束**

每次 Apple 更新审核指南（一般每年 WWDC 和年初各一次）后，本文档需同步更新。
每次被拒后，对应章节补充新的应对话术与技术措施。
