---
name: 对齐 SilenceCut 修复 ATT
overview: 把 FIREprint 的 ATT 触发策略对齐到"在同一台真机上已经工作的 SilenceCut"版本：去掉只触发一次的锁、放宽前台判定、加一条备份触发路径，并抑制 GoogleMobileAds 在 ATT 前的测量层预启动。
todos:
  - id: rewrite_att_logic
    content: 改写 AppDependencies 的 ATT 触发逻辑：去掉 attTask 一次性锁，只等 foregroundActive scene，废弃 applicationState 硬依赖，sleep 降到 600ms（对齐 SilenceCut）
    status: completed
  - id: add_bootstrap_path
    content: 在 FIREprintApp.swift 的 WindowGroup.task 里加一条启动主路径触发 ATT（对齐 SilenceCutApp.swift bootstrap），保留 didBecomeActiveNotification 作为兵底
    status: completed
  - id: delay_gma_measurement
    content: 在 Info.plist 加 GADDelayAppMeasurementInit = YES，防止 GoogleMobileAds 测量层在 ATT 前预启动导致弹窗静默
    status: completed
  - id: build_and_verify
    content: xcodebuild 编译验证通过，用户真机 clean install 后确认 ATT 弹出
    status: completed
  - id: commit
    content: "提交 fix(att): align cold-start ATT trigger with SilenceCut's working strategy"
    status: completed
isProject: false
---

## 背景：SilenceCut 能弹、FIREprint 不弹的根因

你确认 SilenceCut 在这台真机上能正常弹 ATT，所以**不是设备、不是 iOS 系统、不是 bundle id 黑名单、不是"限制广告跟踪"总开关**的问题。问题在 FIREprint 自己的代码里，**4 处**关键差异（从高到低可能性）：

### 差异 1: `attTask` 是"一次性锁"，首次失败后同 session 内无法重试

`[AppDependencies.swift]` 中：

```swift
func startATTRequestIfNeeded() {
    guard attTask == nil else { return }   // <-- 进程级一次性锁
    attTask = Task { await self?.requestATTAuthorization() }
}
```

若首次 `didBecomeActive` 触发时 iOS 静默丢弃（`ATT returned=0`），整个 session 就**再也不会重试**。

SilenceCut 没有这个锁，而且有**两条触发路径**（bootstrap `.task` + 首页 `onHomeReady`），多次机会命中正确时机。

### 差异 2: 前台判定过严

FIREprint 要求：`applicationState == .active` **AND** `foregroundActive scene`。

SilenceCut 注释明确写道：**故意不依赖 `applicationState`**（iPadOS 26 上不准），只看 scene 级 `foregroundActive`。

### 差异 3: 触发路径只有 `didBecomeActiveNotification` 一个

SilenceCut 是 `.task { await requestATT... }` + 首页 `onAppear` 两条路径，FIREprint 只剩 notification 一条。如果第一个 notification 来得太早或被 SwiftUI 生命周期吞掉，就没了。

### 差异 4: AdMob 可能在 ATT 前预启动测量层

- FIREprint 用 **AdMob + `MobileAds.shared.start()`**，`Info.plist` 有 `GADApplicationIdentifier` 但**没有** `GADDelayAppMeasurementInit`，GMA 链接阶段就可能起一些测量。
- SilenceCut 用 **TopOn**，**不**调用 `MobileAds.shared.start()`。
- 如果 GMA 测量层在 ATT 前已启动，iOS 会认为"已有数据收集行为"，**静默吞掉 ATT 弹窗**（这是 Google 官方 FAQ 的已知陷阱）。

### 关于"debug 按钮看不到"

commit `cbac61b` 已经：
- 去掉了 `#if DEBUG` 包装
- 在 `project.yml` 里补了 `SWIFT_ACTIVE_COMPILATION_CONDITIONS: DEBUG`
- 重新 `xcodegen generate` 并 `xcodebuild` 成功

如果你用 Xcode Run（⌘R）把 **commit `cbac61b` 之后** 的版本真正灌到手机上，按钮**必然**会出现（因为已经无条件显示）。看不到只可能是：手机上还是旧 build / 没 clean build / 装错 bundle。

---

## 修复方案：3 步对齐 SilenceCut 的能工作实现

### Step 1: 改写 ATT 触发逻辑（对齐 SilenceCut）

修改 [Projects/FIREprint_App/ios_workspace/FIREprint/App/AppDependencies.swift](Projects/FIREprint_App/ios_workspace/FIREprint/App/AppDependencies.swift)：

1. 把 `attTask` 的一次性锁改成"只锁正在进行中的 request"：
   - 若 `trackingAuthorizationStatus != .notDetermined` → 直接 return（已决定就不再弹）
   - 若有正在 await 的 `attTask` → 等它
   - 否则（包括上一次已结束但 status 仍 `.notDetermined` 的情况）→ 启动新 task
2. `requestATTAuthorization()` 里**去掉对 `applicationState == .active` 的等待**，只等 `hasForegroundActiveKeyScene()`（对齐 `SilenceCutApp.swift` 121–153 行）。
3. 把 `Task.sleep` 从 1s 降回 600ms（SilenceCut 线上参数）。

### Step 2: 补一条"启动主路径"的备用触发

修改 [Projects/FIREprint_App/ios_workspace/FIREprint/App/FIREprintApp.swift](Projects/FIREprint_App/ios_workspace/FIREprint/App/FIREprintApp.swift) 的 `WindowGroup.task`：

- 保留现有的 `requestPrivacyConsent()` 流程
- 但把 ATT 调用**下沉到 bootstrap 的独立 `Task`** 里，和 SilenceCut 415–424 行一致：
  ```swift
  Task {
      await DeviceRegion.configure()
      await dependencies.startATTRequestAndWait()   // 新方法，幂等、可重试
      await dependencies.requestUMPConsent()
      await dependencies.initializeAds()
  }
  ```
- 保留 `onReceive(UIApplication.didBecomeActiveNotification)` 作为第二条兜底路径（两条路径都指向同一个幂等 `startATTRequestAndWait`）。

### Step 3: 抑制 GoogleMobileAds 测量层预启动

修改 [Projects/FIREprint_App/ios_workspace/FIREprint/SupportingFiles/Info.plist](Projects/FIREprint_App/ios_workspace/FIREprint/SupportingFiles/Info.plist)：

```xml
<key>GADDelayAppMeasurementInit</key>
<true/>
```

这是 Google 官方推荐的做法：**延迟 GMA 测量层，直到显式 `MobileAds.start()`**，给 ATT 留出弹窗窗口。

（可选）同时把 `SKAdNetworkItems` 从 SilenceCut 复制过来，纯归因配置，不影响 ATT 但后续上架需要。

---

## 验证步骤（改完由你在真机跑）

1. 删除手机上现有 FIREprint（彻底清 ATT 状态）
2. Xcode → Product → Clean Build Folder（⇧⌘K）
3. ⌘R Run 到真机
4. **首次启动 3 秒内**应弹出 ATT 弹窗
5. 如果仍不弹，打开 App 内 Settings 页 → 点红色盾牌 "DEBUG: Request ATT now"：
   - 若此时弹出 → 确认是"时机"问题，日志看 `hasForegroundActiveKeyScene` 值
   - 若此时也不弹 → 问题在 SDK 预启动层，`GADDelayAppMeasurementInit` 生效后应修复

---

## 不做的事

- 不改 UMP 流程（UMP 本身你确认过能跑）
- 不改 `AppDelegate`（SwiftUI scene app 里它本来就不回调 `didBecomeActive`）
- 不动 Fastlane / metadata / 隐私 URL（那些已经搞定并提交了）