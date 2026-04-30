---
name: split-billing-iap-ads
overview: 把 Shared/MyAppBilling 拆分为纯付费的 MyAppIAP 和独立的 MyAppAds 两个 library；InnerRelease 只依赖 MyAppIAP，SilenceCut 同时依赖两者。拆分保持向后兼容，现有业务逻辑零改动。
todos:
  - id: 1-create-myappads
    content: 新建 Shared/Sources/MyAppAds 目录，将 MyAppBilling/Ads/* 全部复制过去；新增 AdError.swift（顶层化）、AdAnalyticsReporter.swift、AdsConfiguration.swift。暂不删除原 Ads 目录。
    status: pending
  - id: 2-create-myappiap
    content: 新建 Shared/Sources/MyAppIAP 目录，迁入 Common (去掉 .ad 依赖) / IAP / Paywall / PaywallUI / Quota / Rating / Monetization / Resources；新增 IAPBillingConfiguration.swift；修改 BillingError 去掉 .ad case；修改 AnalyticsEventReporter 去掉 reportAdEvent。
    status: pending
  - id: 3-update-package-swift
    content: 更新 Shared/Package.swift：移除 MyAppBilling target/product/testTarget；新增 MyAppIAP + MyAppAds 两个 target + product + testTarget。拆分 Tests/MyAppBillingTests 到 MyAppIAPTests / MyAppAdsTests。
    status: pending
  - id: 4-migrate-silencecut
    content: 更新 SilenceCut：Package.swift 依赖 MyAppBilling → MyAppIAP + MyAppAds；批量替换 import MyAppBilling → import MyAppIAP（仅真正用 Ad 的文件追加 import MyAppAds）；SilenceCutBillingConfig.swift 拆成 iapConfiguration 和 adsConfiguration 两段注入。
    status: pending
  - id: 5-migrate-innerrelease
    content: "更新 InnerRelease project.yml 依赖：MyAppCore 基础上追加 product: MyAppIAP；重新 xcodegen generate；确认编译通过（暂不引入 MyAppAds）。"
    status: pending
  - id: 6-delete-old
    content: 确认两个 App 都能编译并运行后，删除 Shared/Sources/MyAppBilling 旧目录和 Tests/MyAppBillingTests 旧目录。
    status: pending
  - id: 7-verify-builds
    content: 最终 build 验证：xcodebuild InnerRelease Debug + SilenceCut Debug，两端均 BUILD SUCCEEDED；swift test 验证 Shared 单元测试通过。
    status: pending
  - id: 8-docs-update
    content: 更新 Docs/详细设计_MyAppBilling.md 与 Docs/Shared_Integration_Guide_4_AI.md，说明 MyAppBilling 已拆为 MyAppIAP + MyAppAds（可简短增补，不做大重写）。
    status: pending
isProject: false
---

## 背景 & 结论

`MyAppBilling` 目前把"付费 (IAP/Paywall/Quota/Rating/Monetization)"和"广告 (Ads)"放在同一个 library target 里。InnerRelease 是纯付费 App，不需要 Ads 抽象层。经过分析，耦合点其实非常小：

- `Ads/*` 只依赖 Foundation / SwiftUI / MyAppCore，**无任何第三方广告 SDK**
- 除 `Ads/*` 自身外，只有 `Common/BillingConfiguration.swift`、`Common/BillingError.swift`、`Common/AnalyticsEventReporter.swift` 三个文件出现 `Ad*` 类型引用
- `Monetization/*` 的枚举里出现了 `freeWithAds` 等命名（语义硬编码），但**不引用任何 `Ad*` 类型**，保留即可（纯付费 App 把 `.freeWithAds` 当"未付费"语义用，无需改动）
- `Paywall/*`、`PaywallUI/*`、`Quota/*`、`Rating/*`、`IAP/*` 完全不引用 `Ad*`

因此采用"拆目录 + 拆 Package target + 配置类解耦"的方式，以最小改动实现 C 方案。

## 目标产物

拆分后 `Shared/Package.swift` 将新增 2 个 library、移除 1 个（`MyAppBilling`）：

- `MyAppIAP`  — 付费核心：IAP / Paywall / PaywallUI / Quota / Rating / Monetization / Common（无 Ad）/ Resources
- `MyAppAds`  — 广告抽象：Ads/* + 对应错误 + 对应 AnalyticsEventReporter 扩展

## 关键设计决策

### 1. `BillingConfiguration` 怎么办

原 `BillingConfiguration` 同时聚合了 IAP 和 Ad 配置，不适合放在任何一边。处理方式：

- 在 `MyAppIAP` 里新增 `IAPBillingConfiguration`（不含 `ad` 字段），这是 InnerRelease 直接使用的
- 在 `MyAppAds` 里独立定义 `AdsConfiguration`（包一层 `AdConfiguration` + `AdPolicy` 等，方便未来扩展）
- SilenceCut 在 app 层各自构造两个 config 分别注入；保留 `IAPBillingConfiguration` 里的 `paywallConfigProvider` / `quota` / `rating` / `monetization` / `analyticsReporter` 字段以保持向后兼容 API

### 2. `BillingError` 怎么办

原 `BillingError` 的 `.ad(AdError)` case 是唯一的 Ads 耦合点：

- `MyAppIAP/Common/BillingError.swift` 保留 `.iap` / `.quota` / `.paywall` 三个 case，移除 `.ad`
- `MyAppAds/AdError.swift` 独立定义 `AdError`（不再嵌套在 Billing 内），同时提供顶层 `AdOperationError` enum 满足需要统一错误码的场景
- SilenceCut 里目前没有直接拼接 `BillingError.ad(...)` 的代码，影响面基本为零

### 3. `AnalyticsEventReporter` 协议怎么办

原协议同时声明 `reportBillingEvent` / `reportAdEvent` / `reportPaywallEvent`：

- `MyAppIAP` 里保留 `AnalyticsEventReporter` 但只含 `reportBillingEvent` + `reportPaywallEvent`
- `MyAppAds` 里新增独立 `AdAnalyticsReporter` 协议（仅 `reportAdEvent`）
- 上层 Analytics 实现同时 conform 两个协议即可

### 4. 文件物理位置

从 `Shared/Sources/MyAppBilling/` 拆分为：

```
Shared/Sources/
├── MyAppIAP/                          (原 MyAppBilling 除 Ads)
│   ├── Common/
│   │   ├── IAPBillingConfiguration.swift    (原 BillingConfiguration 去掉 ad 字段)
│   │   ├── BillingError.swift                (去掉 .ad case)
│   │   └── AnalyticsEventReporter.swift      (去掉 reportAdEvent)
│   ├── IAP/            (原样迁入)
│   ├── Paywall/        (原样迁入)
│   ├── PaywallUI/      (原样迁入)
│   ├── Quota/          (原样迁入)
│   ├── Rating/         (原样迁入)
│   ├── Monetization/   (原样迁入)
│   └── Resources/      (原样迁入)
└── MyAppAds/                          (全新 target)
    ├── Ads/            (原 MyAppBilling/Ads/* 全部迁入)
    ├── Common/
    │   ├── AdError.swift                     (顶层化)
    │   └── AdAnalyticsReporter.swift         (新协议)
    └── AdsConfiguration.swift                (聚合 config)
```

`Shared/Tests/MyAppBillingTests/` 按照类似方式拆成 `MyAppIAPTests/` + `MyAppAdsTests/`（现有 `IAPAuditLoggerTests` 归 IAP、目前无 Ad 测试）。

### 5. 依赖方向

```mermaid
graph LR
  MyAppCore --> MyAppIAP
  MyAppCore --> MyAppAds
  MyAppIAP -.无依赖.-> MyAppAds
  MyAppAds -.无依赖.-> MyAppIAP
```

`MyAppIAP` 与 `MyAppAds` 互不依赖，都仅依赖 `MyAppCore`。

## 迁移影响矩阵

- **InnerRelease** (`[Projects/InnerRelease_App/ios_workspace/project.yml](Projects/InnerRelease_App/ios_workspace/project.yml)`)：在现有 `- package: Shared / product: MyAppCore` 之外加 `product: MyAppIAP`；不依赖 `MyAppAds`
- **SilenceCut** (`[Projects/SilenceCut_App/ios_workspace/SilenceCut/Package.swift](Projects/SilenceCut_App/ios_workspace/SilenceCut/Package.swift)`)：把原本 `MyAppBilling` 依赖替换为 `MyAppIAP` + `MyAppAds` 两个 product
  - `[SilenceCutBillingConfig.swift](Projects/SilenceCut_App/ios_workspace/SilenceCut/Sources/SilenceCut/App/SilenceCutBillingConfig.swift)`：`import MyAppBilling` → `import MyAppIAP` + `import MyAppAds`
  - 其他 6-10 个业务文件批量 `import MyAppBilling` → `import MyAppIAP`（大部分只用到 IAP/Paywall/Quota/Monetization），只有真正用到 `AdManager`/`AdPolicy` 的文件另加 `import MyAppAds`
- **Shared 文档**：`[Docs/详细设计_MyAppBilling.md](Docs/详细设计_MyAppBilling.md)` 和 `[Docs/Shared_Integration_Guide_4_AI.md](Docs/Shared_Integration_Guide_4_AI.md)` 轻量更新（可留到第 5 步一起做）

## 实施顺序

每一步都要**可编译**（SilenceCut 不能因 Shared 拆分而长时间坏掉）。采取"先加新、验证、再切换、最后删旧"的策略。

## 不做的事

- 不拆分 `MonetizationState`：`freeWithAds` 等枚举值保持原样，InnerRelease 用 `.trial` / `.subscriber` / `.freeWithAds`（语义：未付费）即可
- 不替换第三方 SDK / 不动 SilenceCut 的 TopOn / Admob 适配代码
- 不改 Resources 本地化字符串（Paywall 文案在 IAP 侧）
- 不保留 `MyAppBilling` 兼容别名（避免长期双名），SilenceCut 一次性切过去
