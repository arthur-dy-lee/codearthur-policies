---
name: Fix AdMob Adaptive Banner
overview: 修改 TopOnBannerView 的 banner 加载逻辑，通过 TopOn SDK 的 extra 参数传入 AdMob 自适应横幅尺寸，解决 "Invalid ad width or height" 错误。
todos:
  - id: add-import
    content: TopOnAdapter.swift 顶部添加 import GoogleMobileAds
    status: completed
  - id: fix-extra
    content: TopOnBannerView.makeUIView() 中传入 AdMob 自适应横幅 extra 参数
    status: completed
  - id: fix-height
    content: banner 高度从固定 50 改为 admobSize.size.height
    status: completed
isProject: false
---

# 修复 AdMob 自适应横幅 (Adaptive Banner) 兼容问题

## 问题

AdMob 新建的 Banner 广告单元默认是**自适应横幅**，但 `TopOnBannerView` 加载时 `extra` 为空，TopOn 的 AdMob adapter 默认以固定 320x50 请求 AdMob，导致 `"Invalid ad width or height"` 错误。

## 关键发现

TopOn SDK 头文件 `ATAdManager+Banner.h` 提供了三个参数：

- `kATAdLoadingExtraBannerAdSizeKey` — TopOn 通用 banner 尺寸
- `kATAdLoadingExtraAdmobBannerSizeKey` — AdMob 自适应宽度
- `kATAdLoadingExtraAdmobAdSizeFlagsKey` — AdMob AdSize flags

[TopOn 官方文档示例](https://newdocs.toponad.com/docs/eoMVXS) 展示了如何使用：

```objc
GADAdSize admobSize = GADCurrentOrientationAnchoredAdaptiveBannerAdSizeWithWidth(viewWidth);
extra = @{
    kATAdLoadingExtraBannerAdSizeKey: [NSValue valueWithCGSize:CGSizeMake(viewWidth, 250)],
    kATAdLoadingExtraAdmobBannerSizeKey: [NSValue valueWithCGSize:admobSize.size],
    kATAdLoadingExtraAdmobAdSizeFlagsKey: @(admobSize.flags)
};
```

## 修改文件

[TopOnAdapter.swift](Projects/SilenceCut_App/ios_workspace/SilenceCutApp/TopOnAdapter.swift)

### 改动 1：添加 import

在文件顶部添加 `import GoogleMobileAds`（CocoaPods 已包含 `Google-Mobile-Ads-SDK`）。

### 改动 2：传入 AdMob 自适应 extra 参数

在 `TopOnBannerView.makeUIView()` 中（当前第 509 行），将 `extra: [:]` 替换为：

```swift
let admobSize = GADCurrentOrientationAnchoredAdaptiveBannerAdSizeWithWidth(viewWidth)
let extra: [String: Any] = [
    kATAdLoadingExtraBannerAdSizeKey: NSValue(cgSize: CGSize(width: viewWidth, height: admobSize.size.height)),
    kATAdLoadingExtraAdmobBannerSizeKey: NSValue(cgSize: admobSize.size),
    kATAdLoadingExtraAdmobAdSizeFlagsKey: NSNumber(value: admobSize.flags)
]
ATAdManager.shared().loadAD(withPlacementID: placementID, extra: extra, delegate: context.coordinator)
```

### 改动 3：banner 高度使用自适应高度

将固定的 `bannerHeight = 50` 替换为 AdMob 自适应高度：

```swift
let admobSize = GADCurrentOrientationAnchoredAdaptiveBannerAdSizeWithWidth(viewWidth)
let bannerHeight = admobSize.size.height
```

## TopOn 后台

- 横幅广告位的"广告源尺寸"保持 **横幅 (320x50)** 即可 — 这只影响 TopOn 内部 waterfall 排序，实际请求 AdMob 的尺寸由代码 extra 参数控制
