---
name: FIREprint iCloud 同步启用
overview: iCloud 容器已在 developer.apple.com 建好并绑定 App ID，现在启用 FIREprint 的 CloudKit 同步与 CKShare 账本分享：补齐 entitlements、翻开 FeatureFlags 开关、重新 xcodegen、编译验证、自动提交。
todos:
  - id: entitlements
    content: "改 FIREprint.entitlements: 加 iCloud container + CloudKit service + aps-environment + ubiquity-kvstore"
    status: completed
  - id: feature_flag
    content: FeatureFlags.iCloudSyncEnabled 改为 true
    status: completed
  - id: xcodegen
    content: 重新运行 mint xcodegen 生成 xcodeproj
    status: completed
  - id: build_verify
    content: xcodebuild 编译验证 FIREprint scheme，修复任何编译错误
    status: completed
  - id: commit
    content: "git add + conventional commit (feat: enable iCloud sync + CloudKit sharing for FIREprint)"
    status: completed
isProject: false
---

## 范围

仅做**最小改动**让 iCloud 同步与 CKShare 分享真正生效。不动业务代码（分享流程本身已全部实现），不清理旧文档。

## 要改的文件（共 2 个）

### 1) `Projects/FIREprint_App/ios_workspace/FIREprint/SupportingFiles/FIREprint.entitlements`

当前是空 `<dict/>`。改为以下 4 个键：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.developer.icloud-container-identifiers</key>
    <array>
        <string>iCloud.com.codearthur.matrixapps.fireprint</string>
    </array>
    <key>com.apple.developer.icloud-services</key>
    <array>
        <string>CloudKit</string>
    </array>
    <key>com.apple.developer.ubiquity-kvstore-identifier</key>
    <string>$(TeamIdentifierPrefix)com.codearthur.matrixapps.fireprint</string>
    <key>aps-environment</key>
    <string>development</string>
</dict>
</plist>
```

要点：
- `aps-environment` 是 CloudKit 实时 silent push 通知必需，缺了会导致多设备同步延迟几分钟。App Store 发布打包时 Xcode 会把 `development` 自动换成 `production`。
- `ubiquity-kvstore-identifier` 为未来可能用 `NSUbiquitousKeyValueStore` 预留，现在无害。
- 容器名与代码里 [CloudKitShareManager.swift L18](Projects/FIREprint_App/ios_workspace/FIREprint/Infrastructure/CloudKitShareManager.swift) 严格一致。

### 2) `Projects/FIREprint_App/ios_workspace/FIREprint/App/FeatureFlags.swift`

```swift
static let iCloudSyncEnabled = false    // 改为 true
```

这个开关一翻，以下代码路径全部激活：
- SwiftData 走 `cloudKitDatabase: .automatic`（[FIREprintApp.swift L43](Projects/FIREprint_App/ios_workspace/FIREprint/App/FIREprintApp.swift)）
- `AppDelegate.userDidAcceptCloudKitShareWith` 接收分享邀请
- `ShareInviteView` 里 CloudKit Sharing Section 显示
- `SharingPermission` 启用 Owner/Member 权限判断

## 执行顺序

1. 切到 agent 模式
2. 写 entitlements 文件
3. 改 FeatureFlags
4. 在 `Projects/FIREprint_App/ios_workspace/` 运行 `~/.mint/bin/xcodegen` 重新生成 `FIREprint.xcodeproj`
5. `xcodebuild` 编译验证（Debug / generic iOS）
6. 编译通过 → `git add` 两个文件 + 可能变化的 `.xcodeproj` → conventional commit

## 本轮不做的事

- 不改代码逻辑（CKShare 流程已齐全）
- 不清理 docs 里旧的容器名占位（`iCloud.com.matrixApps.FIRE` / `iCloud.com.codearthur.FireUp`），后续要做可独立任务
- 不写真机测试步骤文档
- 不动 UI / 文案

## 真机验证指引（改完后你自己做）

- **单 iCloud 账号同步**：两台设备都登同一 Apple ID → 新装 FIREprint → 设备 A 建账本记一笔 → 等 30s 内设备 B 看到
- **跨 Apple ID CKShare**：设备 A（账号 A）→ 设置 → 分享邀请 → 对某账本点"开始共享" → 弹系统面板选邀请方式 → 发给账号 A2 → 设备 B（账号 A2）点击链接 → 自动回到 FIREprint → 在账本列表中看到共享账本并可编辑