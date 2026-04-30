---
name: 修复 FIREprint 冷启动 ATT 弹窗不出现
overview: ""
todos:
  - id: reorder_task
    content: 重排 FIREprintApp.swift 的 .task：ATT/UMP/广告优先，loadCurrentUserID 移入独立后台 Task
    status: completed
  - id: timeout_cloudkit
    content: 给 SharingPermission.loadCurrentUserID 加 5s 超时兜底
    status: completed
  - id: att_log
    content: 在 requestATTAuthorization 入口添加 "ATT enter" 日志
    status: completed
  - id: build
    content: xcodebuild 编译验证通过
    status: completed
  - id: verify
    content: 真机删重装验证：ATT 弹窗 <2s 内出现，Console 可见 FIREprint.Privacy 日志
    status: pending
  - id: commit
    content: "提交 fix: don't let CloudKit hang block ATT prompt on cold start"
    status: completed
isProject: false
---

# 修复 FIREprint 冷启动 ATT 弹窗不出现

## 根因
`.task` 先 await 无超时的 `SharingPermission.loadCurrentUserID()`（CloudKit 网络请求），删重装冷启动时可能 hang 5-30 秒，把 ATT 调用推迟到 app 可能已离开前台，系统静默忽略。

## 三个改动

### 改动 1：重排 `.task`（核心修复）
文件：[Projects/FIREprint_App/ios_workspace/FIREprint/App/FIREprintApp.swift](Projects/FIREprint_App/ios_workspace/FIREprint/App/FIREprintApp.swift) 的 `.task` 块

- 将 `requestPrivacyConsent` 和 `initializeAds` 前置
- `SharingPermission.loadCurrentUserID()` 改为独立 `Task { ... }` 并行跑，不阻塞隐私链
- `monetizationStateProvider.currentState()` 保持 await（轻量、本地读取）
- `observeTransactions` for-await 循环保持不变

### 改动 2：`loadCurrentUserID()` 加 5s 超时
文件：[Projects/FIREprint_App/ios_workspace/FIREprint/Infrastructure/SharingPermission.swift](Projects/FIREprint_App/ios_workspace/FIREprint/Infrastructure/SharingPermission.swift) 第 33-41 行

用 `withTaskGroup` 竞速 5 秒超时包裹 `ckContainer.userRecordID()`。超时/失败时 currentUserID getter 自动回退到 device vendor ID。

### 改动 3：ATT 入口加日志
`requestATTAuthorization` 的 guard 之前加 "ATT enter: status=X" 日志，category `FIREprint.Privacy`，便于下次 Console 定位。

## 验证
1. 真机彻底删除 FIREprint
2. Xcode Run 到真机
3. Console 过滤 `FIREprint.Privacy` 看到 "ATT enter" 然后 "ATT finished"
4. 屏幕看到 Apple 原生"允许追踪"弹窗

如仍不弹：iPhone 设置 → 隐私与安全 → 跟踪 → 顶部总开关需要打开。

## 不做的事
- DeviceRegion.configure 位置不变
- UMP CN 跳过 / 超时逻辑不变
- "管理隐私选择"行条件显示不变（用户已确认保持现状）

## 风险
启动头 1-2 秒若用户立即进共享账本，`SharingPermission.currentUserID` 可能返回 fallback device vendor ID 而非 CloudKit 真实 ID。这是现有 getter 的既定 fallback 行为，本改动不引入新问题。

## 提交信息
```
fix(FIREprint): don't let CloudKit hang block ATT prompt on cold start

SharingPermission.loadCurrentUserID() makes an un-timeout CKContainer
request which can hang 5-30s on fresh install, pushing the ATT prompt
past the foregroundActive window. Apple's ATT system silently ignores
requests once the app backgrounds.

- Move ATT/UMP/ad init to the front of the .task chain
- Run CloudKit user ID load in an independent background Task
- Add 5s timeout to loadCurrentUserID as defense in depth
- Log ATT entry state for future diagnosis
```