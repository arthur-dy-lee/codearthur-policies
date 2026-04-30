---
name: SilenceCut广告接入文档
overview: 合并现有 docs/ad/SilenceCut_Ads_Integration_Reference_for_AI.md 与新整理的实现细节，修正旧文档中与代码不符之处，输出单一合并文档到 Projects/SilenceCut_App/docs/SilenceCut_广告接入方案.md。
todos:
  - id: write-merged
    content: 新建 Projects/SilenceCut_App/docs/SilenceCut_广告接入方案.md，写入合并后的完整内容
    status: pending
  - id: update-old-pointer
    content: 更新 Projects/SilenceCut_App/docs/ad/README.md，在顶部加一行指向新文档的 pointer（旧文件作为遗留保留）
    status: pending
isProject: false
---

# SilenceCut 广告接入文档 — 合并计划

## 目标文件

主输出：新建 [`Projects/SilenceCut_App/docs/SilenceCut_广告接入方案.md`](Projects/SilenceCut_App/docs/SilenceCut_广告接入方案.md)

辅助改动：更新 [`Projects/SilenceCut_App/docs/ad/README.md`](Projects/SilenceCut_App/docs/ad/README.md) 在顶部加一行 pointer，说明主文档已迁移到 `docs/SilenceCut_广告接入方案.md`。旧的 `ad/SilenceCut_Ads_Integration_Reference_for_AI.md` 和 `ad/ads_integration_checklist.yaml` **不删除**（避免破坏可能的外部引用），但通过 README pointer 告知 AI 以新文档为准。

## 旧文档 vs 代码实际状态 — 修正项清单

对 [`ad/SilenceCut_Ads_Integration_Reference_for_AI.md`](Projects/SilenceCut_App/docs/ad/SilenceCut_Ads_Integration_Reference_for_AI.md) 的内容逐条校验后，需要修正：

### 要去掉 / 更正的内容

- **去掉**「海外：`GoogleAdMobAdapter`」的说法。SilenceCut 工程里**不存在** `GoogleAdMobAdapter.swift`（唯一一份在 FireUp）。`Podfile` 自己写得很清楚：`TopOn ad mediation — unified entry for all regions.`
- **去掉**「根据地区选择 adapter」「按地区选择 adapter」的所有描述。[`SilenceCutApp.swift`](Projects/SilenceCut_App/ios_workspace/SilenceCutApp/SilenceCutApp.swift) 的 `initializeAds()` 无条件使用 `TopOnAdapter`，`DeviceRegion` 只用来决定是否跳过 UMP。
- **更正**「海外：UMP + ATT；中国区跳过该流程」——实际上 **ATT 对所有地区都请求**，只有 **UMP (GDPR)** 在中国大陆跳过。
- **去掉**「`Google-Mobile-Ads-SDK` 是 TopOn AdMob adapter 的 manual download」这层理解——Podfile 里 `pod 'Google-Mobile-Ads-SDK'` 就是通过 CocoaPods 引入的运行时依赖。

### 要保留（有价值）的内容

- 分层原则（Shared 抽象 vs App 实现）
- TopOn Pod 名迁移历史（v6.3.x `AnyThinkiOS` → v6.5 `TPNiOS` + 独立 adapter pods，Swift 仍 `import AnyThinkSDK`）
- AI 决策硬约束（不要把广告 ID / rollout 放进 Shared；Shared 不直接引入广告 SDK）
- 最小验收标准（banner 能渲染、rewarded 回调可观测、付费免广告、失败可重试）

## 合并后文档结构

### 0. 结论先行（AI 快读版）

- **唯一 adapter**：`TopOnAdapter`（全地区统一入口，中国大陆和海外都走它）
- **变现链路**：`.freeWithAds` 展示 Banner / 导出中 Banner / 激励视频去水印；`.trial` / `.subscriber` 免广告
- **插屏**：代码路径预留，但 Placement ID 未配置，实际不投放
- **初始化触发点**：首页 `onHomeReady`（在 ATT 弹窗闭合之后）
- **Shared vs App 分层**：Shared 只放抽象协议；具体 SDK、Placement ID、rollout 全部在 App 工程

### 1. 核心文件索引

| 文件 | 职责 |
|---|---|
| [`SilenceCutApp/SilenceCutApp.swift`](Projects/SilenceCut_App/ios_workspace/SilenceCutApp/SilenceCutApp.swift) | 启动时序、ATT、UMP、`initializeAds`、重试、首页广告管线 |
| [`SilenceCutApp/TopOnAdapter.swift`](Projects/SilenceCut_App/ios_workspace/SilenceCutApp/TopOnAdapter.swift) | TopOn SDK 桥接、Banner/Interstitial/Rewarded delegate、`TopOnBannerView` |
| [`SilenceCut/App/SilenceCutBillingConfig.swift`](Projects/SilenceCut_App/ios_workspace/SilenceCut/Sources/SilenceCut/App/SilenceCutBillingConfig.swift) | 所有 ID 常量、`AdPolicy`、`TopOnNetwork` rollout |
| [`SilenceCut/Views/HomeView.swift`](Projects/SilenceCut_App/ios_workspace/SilenceCut/Sources/SilenceCut/Views/HomeView.swift) | 首页底部 Banner |
| [`SilenceCut/Views/ExportView.swift`](Projects/SilenceCut_App/ios_workspace/SilenceCut/Sources/SilenceCut/Views/ExportView.swift) | 导出进度 Banner、激励视频去水印 UI |
| [`SilenceCut/ViewModels/ExportViewModel.swift`](Projects/SilenceCut_App/ios_workspace/SilenceCut/Sources/SilenceCut/ViewModels/ExportViewModel.swift) | 激励视频奖励状态机、fallback 逻辑 |
| [`SilenceCutApp/Info.plist`](Projects/SilenceCut_App/ios_workspace/SilenceCutApp/Info.plist) | `GADApplicationIdentifier`、`NSUserTrackingUsageDescription` |
| [`Podfile`](Projects/SilenceCut_App/ios_workspace/Podfile) | TopOn pods + Google-Mobile-Ads-SDK |
| [`Shared/Sources/MyAppBilling/Ads/AdManager.swift`](Shared/Sources/MyAppBilling/Ads/AdManager.swift) | 通用 `AdManager` |
| [`Shared/Sources/MyAppBilling/Ads/AdPolicy.swift`](Shared/Sources/MyAppBilling/Ads/AdPolicy.swift) | `AdPolicy` 数据结构 |
| [`Shared/Sources/MyAppBilling/Ads/DefaultAdPolicyProvider.swift`](Shared/Sources/MyAppBilling/Ads/DefaultAdPolicyProvider.swift) | 策略执行 |
| [`Shared/Sources/MyAppBilling/Ads/AdConfiguration.swift`](Shared/Sources/MyAppBilling/Ads/AdConfiguration.swift) | `AdConfiguration`、`resolvedAdUnitIDs` |
| [`Shared/Sources/MyAppBilling/Ads/AdNetworkAdapter.swift`](Shared/Sources/MyAppBilling/Ads/AdNetworkAdapter.swift) | `AdNetworkAdapter` 协议、`BannerAdViewProvider` |

### 2. 分层规范（保留自旧文档，已校正）

**Shared（抽象层）**
- `AdNetworkAdapter` 协议、`BannerAdViewProvider` 协议
- `AdManager` 通用管理器
- `AdConfiguration` / `AdPolicy` 数据结构
- **不放**：具体 SDK 调用、广告位 ID、rollout 开关、平台特定依赖

**App（实现层）**
- `TopOnAdapter`（对接 TopOn SDK）
- `SilenceCutBillingConfig`（ID 常量 + `TopOnNetwork` + `AdPolicy`）
- `Podfile`（TopOn + Google Mobile Ads SDK）
- Info.plist（`GADApplicationIdentifier`、ATT usage 描述）

### 3. SDK 依赖（Podfile）

**注意：TopOn v6.5 起更换了 pod 名（旧 `AnyThinkiOS` → 新 `TPNiOS` + 独立 adapter pods），Swift import 模块名不变，仍为 `import AnyThinkSDK`。**

当前（SilenceCut 实际使用）：

```ruby
pod 'TPNiOS', '~> 6.5'              # 核心 SDK (当前 6.5.43)
pod 'TPNMediationTTAdapter'         # Pangle/CSJ 穿山甲 (Ads-CN-Beta 7.5.0.1)
pod 'TPNMediationFacebookAdapter'   # Meta Audience Network (6.21.1)
pod 'TPNMediationApplovinAdapter'   # AppLovin (13.6.0)
pod 'Google-Mobile-Ads-SDK'         # TopOn AdMob adapter 的运行时依赖
# Phase 2（已注释）：
# pod 'TPNMediationGDTAdapter'      # 优量汇
# pod 'TPNMediationBaiduAdapter'    # 百青藤
```

### 4. ID 与 Placement 配置

**TopOn**
- App ID: `h69bffe180b6c3`
- App Key: `a374b68dc8321591cd989b42dca9f561e`

**GAD Application ID**（TopOn 的 AdMob adapter 需要在 Info.plist 注册）
- `ca-app-pub-5101769973032466~7722840946`

**TopOn Placement IDs**（`SilenceCutBilling.topOnPlacementIDs` + `topOnExportBannerPlacementID`）

| 广告位 | 格式 | Placement ID | 用途 |
|---|---|---|---|
| 横幅1 | Banner | `n69bffe51e7e06` | 首页底部常驻 Banner |
| 横幅2 | 内联自适应 Banner | `n69bffe5269bcb` | 导出进度页底部 Banner |
| 激励1 | Rewarded | `n69bffe52aab9f` | 去除水印激励视频 |
| —— | Interstitial | **未配置** | `"export_complete"` 插屏路径预留但当前不投放 |

**媒体侧网络 Rollout**（`topOnNetworkRollout`，仅本地标记，真实瀑布由 TopOn 后台控制）

| 网络 | 本地开关 |
|---|---|
| Google (AdMob) | 开 |
| 穿山甲 (Pangle) | 开 |
| Meta | 关 |
| AppLovin | 关 |

### 5. 广告策略（AdPolicy）

```swift
AdPolicy(
    bufferDays: 0,                         // 无安装缓冲期
    interstitialMinIntervalSeconds: 60,
    interstitialMaxPerHour: 3,
    bannerAlwaysOn: true,                  // Banner 不受 bufferDays 约束
    rewardedVideoUnlimited: false,
    adFreeStates: [.trial, .subscriber]    // 这两种状态免广告
)
```

**执行点**：
- `DefaultAdPolicyProvider.shouldShowAd` — adFreeStates / bannerAlwaysOn / bufferDays 判断
- `AdManager` 内部频控 — 仅对 `interstitial` 生效（最小间隔 + 每小时上限）
- `AdManager.showRewardedAd` — 只检查 `userState.shouldShowAds`，不走 `shouldShowAd`（激励由用户主动触发）

`SilenceCutBilling.isPaidUser`：只有 `.freeWithAds` 视为免费用户需要看广告，其余状态（`.trial` / `.subscriber` / `.lifetime` 等）都视为付费。

### 6. 初始化时序（含 ATT 时机，满足 Apple Guideline 2.1）

```
App 冷启动
 └─ SilenceCutApp.body.task → bootstrapIfNeeded()
      ├─ 快速路径：monetizationState 加载（本地 StoreKit，无追踪）
      └─ 后台 Task（隐私链，按顺序 await）：
           ① DeviceRegion.configure()             — 无追踪
           ② ConnectivityMonitor.startMonitoring() — 无追踪
           ③ requestATTAuthorizationWhenReady()   ← ATT 弹窗
              · 最多等 10s 直到有 foreground-active key scene
              · 再 sleep 600ms 让 UI 稳定
              · ATTrackingManager.requestTrackingAuthorization()
           ④ requestPrivacyConsent()              — UMP (GDPR)；中国大陆跳过
           ⑤ ExportNotificationService.requestAuthorization()

首页 SilenceCutAppView 出现 → onHomeReady
 └─ runHomeReadyAdPipeline()
      ├─ requestATTAuthorizationWhenReady()        — 幂等兜底，已定则立即返回
      └─ initializeAds()
           ├─ adManager.setUserStateProvider(...)
           ├─ 读/写 silencecut_install_date (UserDefaults)
           ├─ TopOnAdapter(appID:, appKey:) 构造
           ├─ 构造 AdConfiguration(adUnitIDs: topOnPlacementIDs, ...)
           ├─ await adManager.initialize(configuration:)
           │    └─ TopOnAdapter.initialize(testMode:)
           │         └─ ATAPI.sharedInstance().start(withAppID:, appKey:)
           ├─ initializedAdFormats = { .banner, .rewardedVideo }
           └─ preload(.banner)                      — 实际 load 由 TopOnBannerView 驱动
 └─ warmUpDeferredAdFormatsAfterUIStable()
      · 6s 后 preload(.interstitial)               — 当前 Placement ID 为空，实际跳过
      · 再 4s 后 preload(.rewardedVideo)

initializeAds 失败时：scheduleAdsRetryIfNeeded()
 └─ 最多 3 次重试（10s / 20s / 30s 间隔）
```

**关键约束**：任何涉及追踪的 SDK（TopOn / AdMob / CSJ）都**不在**启动主路径初始化，而是在 `runHomeReadyAdPipeline` 内、`await requestATTAuthorizationWhenReady()` **之后**才触发 `adManager.initialize`。这是符合 Apple Guideline 2.1 的关键：权限请求必须早于任何可用于跟踪的数据采集。

### 7. 各页面广告位对照

| 页面 | placement 字符串 | 格式 | Placement ID 来源 | 触发条件 |
|---|---|---|---|---|
| `HomeView` 底部 | `"home_bottom"` | Banner | `topOnPlacementIDs[.banner]` → `n69bffe51e7e06` | 非付费用户，`.onAppear` 后 1s |
| `ExportView` 进度中 | `"export_progress"` | 内联自适应 Banner | `topOnExportBannerPlacementID` → `n69bffe5269bcb` | 非付费用户 + 导出进行中 |
| `ExportView` 完成页 | `"remove_watermark"` | Rewarded | `topOnPlacementIDs[.rewardedVideo]` → `n69bffe52aab9f` | 用户点击"看广告去水印" |
| `ExportView` 弱引导 | `"export_complete"` | Interstitial | **未配置** | 当前不投放 |

### 8. TopOnAdapter 实现要点

- **Banner**：`load()` 只标记 placement ID 即返回 `true`，真正的 `ATAdManager.loadAD` 发生在 `TopOnBannerView.makeUIView`（UIViewRepresentable）内。`retrieveBannerView` 在 `didFinishLoadingAD` 回调后 attach 到 container。
- **Rewarded 自动预加载**：`rewardedVideoDidClose` 触发后通过 `onPreloadNext` 立即再 `loadRewarded`，保证下次即取即用。
- **插屏展示后自动预加载**：`showInterstitial` 返回前启动后台 `loadInterstitial`。
- **30 秒超时保护**：`showInterstitial` 使用 `TaskGroup` 做超时兜底。
- **自定义关闭按钮**：`TopOnCloseOverlayWindow`（`UIWindow`, `windowLevel: .alert + 1`）在激励视频上层覆盖一个关闭按钮，hit-test 只命中按钮，其余事件穿透到广告。
- **SDK 拼写陷阱**：`rewardedVideoDidRewardSuccess(forPlacemenID:)`（SDK 里 `Placement` 少了一个 `t`），Swift 必须精确匹配 ObjC selector。

### 9. AdManager 公共 API（Shared）

```swift
// AdManagerProtocol
func initialize(configuration: AdConfiguration) async
func preload(format: AdFormat) async throws
func showAd(format: AdFormat, placement: String) async -> AdEvent
func canShowAd(format: AdFormat) async -> Bool
func bannerView(placement: String) async -> AnyView?
func bannerView(placement: String, adUnitID: String) async -> AnyView?
func mediumRectangleBannerView(placement: String) async -> AnyView?
func inlineAdaptiveBannerView(placement: String, adUnitID: String) async -> AnyView?
func showRewardedAd(placement: String) async -> AdEvent

// AdManager 专有
public init()
public func setUserStateProvider(_ provider: @escaping @Sendable () async -> MonetizationState)
```

`AdFormat`: `.banner` / `.interstitial` / `.rewardedVideo` / `.native`

### 10. 新 App 迁移步骤（AI 执行版）

1. 新 App 工程创建：
   - `<App>App/TopOnAdapter.swift`（从 SilenceCut 拷骨架，替换日志 category）
   - `<App>/Sources/<App>/App/<App>BillingConfig.swift`（替换 App ID / Key / Placement IDs）
2. `<App>BillingConfig` 必含：
   - `topOnAppID` / `topOnAppKey`
   - `topOnPlacementIDs: [AdFormat: String]`
   - `topOnExportBannerPlacementID`（如果有导出类场景）
   - `TopOnNetwork` 枚举 + `topOnNetworkRollout: [TopOnNetwork: Bool]`
   - `AdPolicy`
3. 启动入口需要的顺序：
   - `DeviceRegion.configure()`
   - 隐私链：`requestATTAuthorizationWhenReady()` → `requestPrivacyConsent()`（UMP，国区跳过）
   - `runHomeReadyAdPipeline()` 在首页 `onHomeReady` 触发
4. `Podfile` 使用新平台：`TPNiOS (~> 6.5)` + `TPNMediation*Adapter` + `Google-Mobile-Ads-SDK`
5. `Info.plist` / `project.yml` 配：
   - `GADApplicationIdentifier`
   - `NSUserTrackingUsageDescription`（英文短句，避免审核员歧义）
   - `SKAdNetworkItems`
6. 验收：国内 / 海外各测一次；Banner 渲染、Rewarded 回调、失败重试、付费状态下无广告。

### 11. AI 决策硬约束

- **不要**把 `TopOnNetwork` rollout / Placement ID / App Key 放进 Shared。
- **不要**在 Shared 里直接 `import AnyThinkSDK` / `import GoogleMobileAds`（ObjC/平台特定依赖只留在 App 工程）。
- **不要**在 ATT 之前初始化 `AdManager` / TopOn SDK / 任何可能触及 IDFA 的代码路径。
- **不要**假设存在 `GoogleAdMobAdapter` —— SilenceCut 只有 `TopOnAdapter`，AdMob 通过 TopOn 聚合。
- **不要**在启动主路径 `await` 广告 SDK 初始化，必须在后台 Task + 允许失败 + 重试。

### 12. 已知缺口 / 待办

- [ ] Interstitial Placement ID 未配置，`"export_complete"` 路径当前是 dead code
- [ ] `testAdUnitIDs` 未传入 `initializeAds`，DEBUG 依赖 AdMob test device ID + TopOn header bidding test mode
- [ ] Meta / AppLovin 本地 rollout 为 `false`，实际开启还需 TopOn 后台瀑布配合
- [ ] 二期计划：`TPNMediationGDTAdapter`（优量汇）、`TPNMediationBaiduAdapter`（百青藤）
- [ ] 中国大陆用户广告体验（CSJ 穿山甲的实际填充、首帧时延）未单独验证

### 13. 最小验收标准（继承自旧文档，保留）

- 存在至少一个可用 adapter（SilenceCut 是 `TopOnAdapter`）
- `adManager.initialize` 走后台路径，不阻塞主交互
- `.freeWithAds` 展示广告，`.trial` / `.subscriber` 不展示
- Banner 至少一个页面可稳定渲染（`HomeView` 底部）
- Rewarded 回调可观测（`.rewardEarned` / `.dismissed` / `.loadFailed`）
- 初始化失败可重试且不卡首屏

## 执行步骤

1. 写入 [`Projects/SilenceCut_App/docs/SilenceCut_广告接入方案.md`](Projects/SilenceCut_App/docs/SilenceCut_广告接入方案.md)（使用上面 0–13 节内容）。
2. 在 [`Projects/SilenceCut_App/docs/ad/README.md`](Projects/SilenceCut_App/docs/ad/README.md) 顶部加一行 pointer：

   ```
   > **主文档已迁移到 [`../SilenceCut_广告接入方案.md`](../SilenceCut_广告接入方案.md)**。本目录下的文件作为历史参考保留；如有冲突以主文档为准。
   ```

3. **不动**旧文件 `ad/SilenceCut_Ads_Integration_Reference_for_AI.md` 和 `ad/ads_integration_checklist.yaml`（避免破坏外部引用；通过 README pointer 告知优先级）。
