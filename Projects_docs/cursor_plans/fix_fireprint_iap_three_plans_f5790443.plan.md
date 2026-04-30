---
name: fix fireprint iap three plans
overview: 让 FIREprint 付费页显示月付/年付/永久买断三档。代码和 storekit 已经写好三款，核心只需把 Configuration.storekit 挂到 Scheme 上，Xcode Debug 就能看到 3 张卡。上线前需在 ASC 补建年付订阅。
todos:
  - id: yml
    content: "project.yml schemes.FIREprint.run 新增 storeKitConfiguration: FIREprint/Configuration.storekit"
    status: completed
  - id: regen
    content: 运行 ~/.mint/bin/xcodegen 重生成 FIREprint.xcodeproj，确认 xcscheme 含 StoreKitConfigurationFileReference
    status: completed
  - id: build
    content: xcodebuild generic/platform=iOS Debug 构建验证通过
    status: completed
  - id: commit
    content: "提交 fix: wire local StoreKit config to FIREprint scheme so paywall shows 3 plans in Debug"
    status: completed
isProject: false
---

## 背景

目标：付费页显示 **月付 + 年付 + 永久买断** 三档。

现状盘点：

| 侧 | 月付 | 年付 | 永久买断 | 说明 |
|---|---|---|---|---|
| [FIREprintBillingConfig.swift](Projects/FIREprint_App/ios_workspace/FIREprint/App/FIREprintBillingConfig.swift) | ✅ | ✅ | ✅ | 三款全有，`iapConfiguration.productIDs` / `PaywallConfiguration.productIDs` 都已包含 |
| [Configuration.storekit](Projects/FIREprint_App/ios_workspace/FIREprint/Configuration.storekit) | ✅ $2.0 | ✅ $18.0 | ✅ $30.0 | 三款全有 |
| Scheme 是否挂 storekit | ❌ | ❌ | ❌ | **问题点 1**：Xcode Debug 没用本地模拟，走真实 ASC |
| App Store Connect | ✅ | ❌ | ✅ | **问题点 2**：ASC 缺年付，TestFlight/上架拿不到 |

所以 Xcode Debug 只展示月付，是因为 StoreKit 去真实 ASC 拉，而 ASC 只有月付是 Approved / Ready 状态。

价格不在此次改动范围：`product.displayPrice` 运行时自动取——Debug 挂 storekit 时取 storekit 模拟值，TestFlight / 上架时取 ASC Price Tier，代码从不写死。

## 本次代码改动（仅 1 处）

### 把 `Configuration.storekit` 挂到 Scheme

文件：[Projects/FIREprint_App/ios_workspace/project.yml](Projects/FIREprint_App/ios_workspace/project.yml)

`schemes.FIREprint.run` 下新增 `storeKitConfiguration`：

```yaml
schemes:
  FIREprint:
    build:
      targets:
        FIREprint: all
    run:
      config: Debug
      storeKitConfiguration: FIREprint/Configuration.storekit
```

xcodegen 会在生成的 `FIREprint.xcscheme` 的 `LaunchAction` 中注入：

```xml
<StoreKitConfigurationFileReference
   identifier = "../FIREprint/Configuration.storekit">
</StoreKitConfigurationFileReference>
```

### 重新生成 Xcode 工程

```bash
cd Projects/FIREprint_App/ios_workspace
~/.mint/bin/xcodegen
```

按项目约定用 mint 版本（brew 版本会生成 Xcode 16 格式）。

### 构建验证

按 CLAUDE.md 规矩，改动后必须编译通过：

```bash
cd Projects/FIREprint_App/ios_workspace
xcodebuild -project FIREprint.xcodeproj -scheme FIREprint \
  -destination 'generic/platform=iOS' -configuration Debug build 2>&1 | tail -30
```

### 运行验证

Xcode 选 FIREprint scheme → Debug 运行 → 打开付费页，应当看到三张卡片（价格取自 storekit 模拟值）：

- `Monthly Pro $2.00 / 月`
- `Yearly Pro $18.00 / 年`
- `Lifetime Pro $30.00 一次性`（排最后，带 "Best Value" 徽章）

## 不改动的部分

- 代码已正确，`FIREprintBillingConfig.swift` 不用动
- Shared 层不动
- storekit 文件里的模拟价（`2.0 / 18.0 / 30.0`）不动，只是 Debug 预览，上线价由 ASC 决定

## 上线前你需要去 ASC 做（非代码工作）

要让 TestFlight 和正式环境也显示 3 档，必须在 App Store Connect 把年付那款产品建出来：

1. ASC → FIREprint → Monetization → In-App Purchases → **+** → 选 **Auto-Renewable Subscription**
2. 订阅组：选已有的 `FIREprint Pro`（和月付放一起）
3. **Product ID**：必须填 `com.codearthur.matrixApps.FIREprint.pro.yearly`（不能错一个字，要和 Swift 常量一致）
4. Subscription Duration：`1 Year`
5. Reference Name：`Yearly Pro`
6. Pricing：按你定的年付价选 Price Tier
7. Localization（至少 `en-US` + `zh-Hans`）：Display Name `Yearly Pro` / `年付 Pro`，Description `Yearly access to all Pro features` / `每年解锁全部 Pro 功能`
8. 上传一张 Paywall 截图（要能看到 Yearly 那张卡）
9. 状态置为 **Ready to Submit**
10. 在下一个 App 版本页面底部 **In-App Purchases** 勾上它，随版本一起提交审核

审核通过后，**TestFlight 和正式上架都会自动显示 3 档**，一行代码都不用改。

## 提交

按项目自动提交规则，改完编译通过后以 `fix: wire local StoreKit config to FIREprint scheme so paywall shows 3 plans in Debug` 提交。
