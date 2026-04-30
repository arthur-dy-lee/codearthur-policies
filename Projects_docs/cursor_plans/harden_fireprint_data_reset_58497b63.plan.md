---
name: Harden FIREprint data reset
overview: 彻底重写 FIREprint 的"清除全部数据"流程:清空 CloudKit 默认 zone、增强 re-seed 容错、补全日志,并给分类图标加 nil/empty 兜底,避免清除后界面出现空白图标。
todos:
  - id: expose-zone-name
    content: 在 CloudKitShareManager 或 LedgerShareService 暴露 defaultCoreDataZoneName 常量
    status: completed
  - id: wipe-default-zone
    content: 在 LedgerShareService 新增 wipeDefaultCoreDataZone(),复用 20s timeout + zoneNotFound=success 模式
    status: completed
  - id: rewrite-delete-all
    content: 重写 LedgerManagerView.deleteAllData():并行 wipe 双 zone、补 Category/Ledger/FIREProfile/AssetSnapshot/UserSettings count 日志
    status: completed
  - id: reseed-fallback
    content: 把 re-seed 块提取成 reseedDefaultData(),JSON 失败时调 reseedMinimalFallback() 插入 3 个兜底分类
    status: completed
  - id: display-icon
    content: Category 新增 displayIcon computed property,作为 icon 空值的统一兜底
    status: completed
  - id: replace-icon-usage
    content: CategoryManagerView.swift 内 7 处 Text(category/parent/child.icon) 改为 .displayIcon
    status: completed
  - id: build-verify
    content: xcodebuild 编译验证 FIREprint target 通过
    status: completed
isProject: false
---

## 目标与范围

只做用户确认的三件事,其它优化以后再说:

1. `deleteAllData()` 新增**默认 CoreData zone** 的 wipe(SwiftData 数据实际所在的 zone)
2. re-seed 加容错 + 完整日志
3. 分类图标 UI 加 empty fallback

## 改动一:新增 `wipeDefaultCoreDataZone()`

**文件:** [`Projects/FIREprint_App/ios_workspace/FIREprint/Infrastructure/LedgerShareService.swift`](Projects/FIREprint_App/ios_workspace/FIREprint/Infrastructure/LedgerShareService.swift)

在现有 `wipeOwnedSharesAndZone()` 下方(第 854 行之后)新增一个并列函数,专门 wipe SwiftData 的默认 zone。复用相同的 20s 超时 + `zoneNotFound` 视为成功的模式:

- Zone ID 常量来自 [`CloudKitShareManager.swift:19`](Projects/FIREprint_App/ios_workspace/FIREprint/Infrastructure/CloudKitShareManager.swift) `ckZoneName = "com.apple.coredata.cloudkit.zone"`
- 先把它暴露成 `public static let defaultCoreDataZoneName`,方便跨文件共享
- 新函数签名:`func wipeDefaultCoreDataZone(timeoutSeconds: TimeInterval = 20) async throws`
- 日志前缀 `wipeDefaultCoreDataZone:`(和现有 `wipeOwnedSharesAndZone:` 风格一致)
- **不**调用 `zoneEnsured = false`(这个字段专属 SharedLedgersZone,默认 zone 由 SwiftData 自管理)

## 改动二:重写 `deleteAllData()`

**文件:** [`Projects/FIREprint_App/ios_workspace/FIREprint/Views/Settings/LedgerManagerView.swift`](Projects/FIREprint_App/ios_workspace/FIREprint/Views/Settings/LedgerManagerView.swift)(第 680-810 行)

### 2.1 云端 wipe 扩展到两个 zone

把现有的单 zone `Task.detached` 改成并行 wipe 两个 zone:

```swift
if FeatureFlags.iCloudSyncEnabled {
    Task.detached(priority: .utility) {
        async let sharedWipe: Void = LedgerShareService.shared.wipeOwnedSharesAndZone()
        async let defaultWipe: Void = LedgerShareService.shared.wipeDefaultCoreDataZone()
        // 分别 try?,任何一个失败不影响另一个 / 本地流程
        do { try await sharedWipe } catch { AppLogger.warning("deleteAllData: SharedLedgersZone wipe failed: \(error.localizedDescription)", category: "FIREprint.Settings") }
        do { try await defaultWipe } catch { AppLogger.warning("deleteAllData: default CoreData zone wipe failed: \(error.localizedDescription)", category: "FIREprint.Settings") }
        AppLogger.warning("deleteAllData: cloud wipe stage done (both zones attempted)", category: "FIREprint.Settings")
    }
}
```

### 2.2 本地删除阶段:补全每类的 count 日志

当前只有 `txCount` 有日志,其它都是静默。给 `Category` / `Ledger` / `FIREProfile` / `AssetSnapshot` / `UserSettings` 五类都加 "fetched N, deleting..." 日志:

```swift
let catCount = (try? modelContext.fetch(catDescriptor).count) ?? 0
AppLogger.warning("deleteAllData: deleting \(catCount) categories", category: "FIREprint.Settings")
```

### 2.3 re-seed 阶段:容错 + 日志

把当前 758-797 行的 seed 块提取成私有方法 `reseedDefaultData(savedOwnerDeviceID:)`,加结构化错误路径:

- JSON 定位 / 读取 / 解析任一失败 → log `error`,调用 `reseedMinimalFallback()` 插入 3 个兜底分类(💰 工资 / 🍱 餐饮 / 🚕 交通)+ 默认账本
- seed 成功 → log `"reseeded N parent categories, M subcategories"`
- 最后 log `"reseed total: N categories, 1 ledger, 1 settings"`

### 2.4 新增 `reseedMinimalFallback()`

仅在 JSON 路径失败时调用,硬编码 3 个父分类(不含子分类)。避免"一片空白"的最坏体验。图标用 emoji,与 JSON 风格一致。

### 2.5 流程总结日志

函数开头和结尾各打一行,包含 Apple ID(脱敏)、耗时、seed 模式(JSON / fallback):

```
deleteAllData: start (iCloudSyncEnabled=true)
deleteAllData: done in 3s (seedMode=json, cloud wipe in flight)
```

## 改动三:分类图标空值兜底

**策略:** 在 `Category` 模型加 computed 属性 `displayIcon`,统一兜底。比在每个 UI 点改 `Text(x.icon.isEmpty ? "📄" : x.icon)` 更干净、更可维护。

### 3.1 [`Projects/FIREprint_App/ios_workspace/FIREprint/Models/Category.swift`](Projects/FIREprint_App/ios_workspace/FIREprint/Models/Category.swift)

在 `fullPath` 下方新增:

```swift
/// Icon with empty-string fallback. Used by all UI sites so that a blank
/// `icon` field (possible after partial reset / CloudKit sync gap) never
/// renders as an invisible glyph.
var displayIcon: String {
    icon.isEmpty ? "📄" : icon
}
```

### 3.2 替换所有 `Text(category.icon)` 等用法

按用户要求先改 [`CategoryManagerView.swift:219`](Projects/FIREprint_App/ios_workspace/FIREprint/Views/Settings/CategoryManagerView.swift)。为了这次修复彻底(否则修了一个页面其它页面还空白),把 **同文件内** 所有同类调用一起改掉:
- `CategoryManagerView.swift` 第 219, 318, 590, 610, 692, 716, 749 行 → `x.displayIcon`
- 第 671 行 `Text(source.icon)` 不改(`source` 是不同类型,本次不扩大范围)

**其它文件不在本次范围内**(CategoryPickerView / ExportView / RecordView / StatsView / TransactionQueryView),等用户确认后续优化再处理。

## 验证步骤

改完后执行:

```bash
cd Projects/FIREprint_App/ios_workspace
xcodebuild -workspace FIREprint.xcworkspace -scheme FIREprint \
  -destination 'generic/platform=iOS' -configuration Debug build 2>&1 | tail -30
```

## 文件改动一览

- [`Projects/FIREprint_App/ios_workspace/FIREprint/Infrastructure/LedgerShareService.swift`](Projects/FIREprint_App/ios_workspace/FIREprint/Infrastructure/LedgerShareService.swift):新增 `wipeDefaultCoreDataZone()` + 暴露常量
- [`Projects/FIREprint_App/ios_workspace/FIREprint/Views/Settings/LedgerManagerView.swift`](Projects/FIREprint_App/ios_workspace/FIREprint/Views/Settings/LedgerManagerView.swift):重写 `deleteAllData()`(拆分子方法、双 zone wipe、re-seed 容错、补日志)
- [`Projects/FIREprint_App/ios_workspace/FIREprint/Models/Category.swift`](Projects/FIREprint_App/ios_workspace/FIREprint/Models/Category.swift):新增 `displayIcon`
- [`Projects/FIREprint_App/ios_workspace/FIREprint/Views/Settings/CategoryManagerView.swift`](Projects/FIREprint_App/ios_workspace/FIREprint/Views/Settings/CategoryManagerView.swift):7 处 `Text(*.icon)` → `Text(*.displayIcon)`

## 范围之外(本次不做)

- 其它 View 文件里的 `Text(*.icon)` 用法(CategoryPickerView, ExportView, RecordView, StatsView, TransactionQueryView)
- `DefaultCategories.json` 是否真被打包进 bundle 的排查
- 多设备数据推回 / CloudKit quota 满的告警 UI
- iCloud 备份层的清理提示
