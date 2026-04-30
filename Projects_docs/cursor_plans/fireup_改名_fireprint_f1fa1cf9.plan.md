---
name: FireUp 改名 FIREprint
overview: 将 iOS 工程从 FireUp 全面改名为 FIREprint，包括：目录/文件重命名、project.yml 配置、Info.plist、Swift 源文件中的字符串和类型名、IAP Product ID、CloudKit Container ID、脚本文件。
todos:
  - id: rename-dir-files
    content: 重命名目录 FireUp/ → FIREprint/ 及 3 个含 FireUp 的 Swift 文件名和 entitlements 文件名
    status: completed
  - id: update-project-yml
    content: 修改 project.yml：name、scheme、target、路径、Bundle ID 全部改为 FIREprint/fireprint
    status: completed
  - id: update-info-plist
    content: 修改 Info.plist：CFBundleDisplayName、CFBundleURLName、4 个 Usage Description 改为 FIREprint
    status: completed
  - id: update-billing-config
    content: 修改 FIREprintBillingConfig.swift：enum 改名、IAP Product ID、PaywallConfig appName 改为 FIREprint
    status: completed
  - id: update-main-app
    content: 修改 FIREprintApp.swift：struct 改名、appName 字符串、fireup_install_date key
    status: completed
  - id: update-paywall-wrapper
    content: 修改 FIREprintPaywallWrapper.swift：struct 和内部引用改名
    status: completed
  - id: update-infra-files
    content: 修改 CloudKitShareManager、KeychainService、SharingPermission 中的硬编码 ID
    status: completed
  - id: update-other-swift
    content: 批量替换其余 17 个 Swift 文件中的 FireUp 字符串（日志 category、UI 文案、注释）
    status: completed
  - id: update-storekit
    content: 修改 Configuration.storekit：productID 和订阅组名改为 FIREprint
    status: completed
  - id: update-scripts
    content: 修改 generate.sh 和 sim_import.sh 中的 Bundle ID 和工程名
    status: completed
  - id: regenerate-xcode
    content: 运行 ./generate.sh 重新生成 Xcode 工程并验证编译
    status: completed
  - id: dev-portal
    content: 在 Apple Developer Portal 手动注册新 App ID、iCloud Container、App Group；在 App Store Connect 新建 App 和 IAP 产品
    status: in_progress
isProject: false
---

# FireUp → FIREprint 全量改名计划

## 变更映射总表

| 类型 | 旧值 | 新值 |
|------|------|------|
| Bundle ID | `com.codearthur.FireUp` | `com.codearthur.matrixapps.fireprint` |
| iCloud Container | `iCloud.com.codearthur.FireUp` | `iCloud.com.codearthur.matrixapps.fireprint` |
| IAP 前缀 | `com.codearthur.matrixApps.FireUp.pro.*` | `com.codearthur.matrixApps.FIREprint.pro.*` |
| Keychain Service | `com.matrixApps.FireUp` | `com.matrixApps.FIREprint` |
| URL Name | `com.matrixApps.FireUp` | `com.matrixApps.FIREprint` |
| UserDefaults Key | `fireup_install_date` | `fireprint_install_date` |
| Display Name | `FireUp` | `FIREprint` |

> **Apple Developer Portal 侧需手动操作（不在代码里改）**：在 Developer Portal 新建 App ID `com.codearthur.matrixapps.fireprint`，新建 iCloud Container `iCloud.com.codearthur.matrixapps.fireprint`。在 App Store Connect 里新建 App 填写 `FIREprint` 并创建新 IAP 产品。

---

## 重要注意事项

- **IAP Product ID 不可改名**，旧 `FireUp.pro.*` 必须在 App Store Connect 创建全新产品 `FIREprint.pro.*`；代码同步改为新 ID
- **UserDefaults Key 改名**（`fireup_install_date` → `fireprint_install_date`）会导致旧用户丢失安装日期记录（影响广告 buffer days 计算）；若需兼容，代码里可做迁移
- **目录重命名**（`FireUp/` → `FIREprint/`）是改名核心，所有 `project.yml` 路径引用需同步

---

## Step 1：目录与文件重命名

重命名以下目录和文件：

- `ios_workspace/FireUp/` → `ios_workspace/FIREprint/`
- `FireUp/App/FireUpApp.swift` → `FIREprint/App/FIREprintApp.swift`
- `FireUp/App/FireUpBillingConfig.swift` → `FIREprint/App/FIREprintBillingConfig.swift`
- `FireUp/Views/Settings/FireUpPaywallWrapper.swift` → `FIREprint/Views/Settings/FIREprintPaywallWrapper.swift`
- `FireUp/SupportingFiles/FireUp.entitlements` → `FIREprint/SupportingFiles/FIREprint.entitlements`

---

## Step 2：[`project.yml`](Projects/FireUp_App/ios_workspace/project.yml) — 5 处改动

```yaml
# 旧 → 新
name: FireUp → name: FIREprint
schemes:
  FireUp: → FIREprint:
    build:
      targets:
        FireUp: all → FIREprint: all
targets:
  FireUp: → FIREprint:
    sources:
      - path: FireUp → path: FIREprint
    settings:
      INFOPLIST_FILE: FireUp/SupportingFiles/Info.plist → FIREprint/SupportingFiles/Info.plist
      CODE_SIGN_ENTITLEMENTS: FireUp/SupportingFiles/FireUp.entitlements → FIREprint/SupportingFiles/FIREprint.entitlements
      PRODUCT_BUNDLE_IDENTIFIER: com.codearthur.FireUp → com.codearthur.matrixapps.fireprint
    resources:
      - path: FireUp/Resources → FIREprint/Resources
```

---

## Step 3：[`Info.plist`](Projects/FireUp_App/ios_workspace/FireUp/SupportingFiles/Info.plist) — 5 处改动

- `CFBundleDisplayName`: `FireUp` → `FIREprint`
- `CFBundleURLName`: `com.matrixApps.FireUp` → `com.matrixApps.FIREprint`
- `NSFaceIDUsageDescription`: `FireUp uses Face ID...` → `FIREprint uses Face ID...`
- `NSMicrophoneUsageDescription`: `FireUp needs microphone...` → `FIREprint needs microphone...`
- `NSSpeechRecognitionUsageDescription`: `FireUp uses speech...` → `FIREprint uses speech...`
- `NSUserTrackingUsageDescription`: `FireUp uses this...` → `FIREprint uses this...`

---

## Step 4：Swift 源文件改动

### 4a. [`FireUpApp.swift`](Projects/FireUp_App/ios_workspace/FireUp/App/FireUpApp.swift) → 重命名为 `FIREprintApp.swift`

- `struct FireUpApp: App` → `struct FIREprintApp: App`
- `appName: "FireUp"` → `appName: "FIREprint"` (AppLockView)
- `fireup_install_date` → `fireprint_install_date`

### 4b. [`FireUpBillingConfig.swift`](Projects/FireUp_App/ios_workspace/FireUp/App/FireUpBillingConfig.swift) → 重命名为 `FIREprintBillingConfig.swift`

- `enum FireUpBilling` → `enum FIREprintBilling`
- IAP Product IDs: `FireUp.pro.*` → `FIREprint.pro.*`
- `struct FireUpPaywallConfig` → `struct FIREprintPaywallConfig`
- URL 参数 `?name=FireUp` → `?name=FIREprint`
- 所有 `FireUpBilling.xxx` 引用 → `FIREprintBilling.xxx`（同文件内）

### 4c. [`FireUpPaywallWrapper.swift`](Projects/FireUp_App/ios_workspace/FireUp/Views/Settings/FireUpPaywallWrapper.swift) → 重命名为 `FIREprintPaywallWrapper.swift`

- `struct FireUpPaywallWrapper` → `struct FIREprintPaywallWrapper`
- `FireUpPaywallConfig()` → `FIREprintPaywallConfig()`

### 4d. [`CloudKitShareManager.swift`](Projects/FireUp_App/ios_workspace/FireUp/Infrastructure/CloudKitShareManager.swift)

- `containerID = "iCloud.com.codearthur.FireUp"` → `"iCloud.com.codearthur.matrixapps.fireprint"`

### 4e. [`KeychainService.swift`](Projects/FireUp_App/ios_workspace/FireUp/Infrastructure/Keychain/KeychainService.swift)

- `serviceName = "com.matrixApps.FireUp"` → `"com.matrixApps.FIREprint"`

### 4f. [`SharingPermission.swift`](Projects/FireUp_App/ios_workspace/FireUp/Infrastructure/SharingPermission.swift)

- `fireup_anonymous_user_id` → `fireprint_anonymous_user_id`

### 4g. 其余 Swift 文件（批量 `FireUp` → `FIREprint` 字符串替换）

含 `FireUp` 字符串的文件（日志 category、注释、UI 硬编码）：
- `CloudSharingView.swift`、`WelcomeView.swift`、`SceneManagerView.swift`、`StatsView.swift`、`Font+Theme.swift`、`FlowListView.swift`、`SettingsView.swift`、`RecordView.swift`、`AppDelegate.swift`、`ProFeatureGateView.swift`、`CategoryManagerVM.swift`、`DashboardVM.swift`、`DashboardView.swift`、`ExportView.swift`、`SimulatorVM.swift`、`ShareInviteView.swift`、`GoogleAdMobAdapter.swift`

---

## Step 5：[`Configuration.storekit`](Projects/FireUp_App/ios_workspace/FireUp/Configuration.storekit)

- 3 处 `productID`: `FireUp.pro.*` → `FIREprint.pro.*`
- `name: "FireUp Pro"` → `"FIREprint Pro"`

---

## Step 6：脚本文件

### [`generate.sh`](Projects/FireUp_App/ios_workspace/generate.sh)

- `PBXPROJ="FireUp.xcodeproj/project.pbxproj"` → `FIREprint.xcodeproj/project.pbxproj`
- `Open FireUp.xcodeproj` → `FIREprint.xcodeproj`

### [`sim_import.sh`](Projects/FireUp_App/sim_import.sh)

- `BUNDLE_ID="com.codearthur.FireUp"` → `"com.codearthur.matrixapps.fireprint"`
- 注释/输出文案 `FireUp` → `FIREprint`

---

## Step 7：重新生成 Xcode 工程

```bash
cd Projects/FireUp_App/ios_workspace
./generate.sh
```

然后在 Xcode 中用新 Bundle ID `com.codearthur.matrixapps.fireprint` 重新 Sign。

---

## Step 8：Apple Developer Portal 手动操作（代码改完后）

按上一轮讨论的 Capabilities 清单，在 Developer Portal 注册：

1. 新建 iCloud Container: `iCloud.com.codearthur.matrixapps.fireprint`
2. 新建 App Group: `group.com.codearthur.matrixapps.fireprint`
3. 注册 App ID: `com.codearthur.matrixapps.fireprint`，关联上述 Container 和 Group
4. App Store Connect 新建 App，填写 `FIREprint`，关联新 Bundle ID
5. 创建新 IAP 产品（`FIREprint.pro.monthly / yearly / lifetime`）

---

## 不改的部分

- `Projects/FireUp_App/` 目录名（改动会影响 `matrixAppsCore/config.yaml` 等大量 Python 侧引用，性价比低）
- `docs/` 下的文档文件名（内容里的 `FireUp` 可选择性更新）
- App icon 图片文件名（`FireUp_v2_1024.png` 只是资源文件名，不影响运行）
