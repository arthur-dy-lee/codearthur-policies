---
name: FIREprint Shared Category Sync
overview: 把 Category 也纳入共享账本跨账号同步：稳定 UUID 的系统预设 + 软删除 + 中英文双语字段 + Transaction 改用 CKReference 挂分类，采用 last-write-wins 合并策略，全局作用域。
todos:
  - id: model-fields
    content: Category.swift 添加 updatedAt/deletedAt/nameZh/nameEn 字段，新增 localizedName 计算属性，所有本地写入点统一刷 updatedAt
    status: completed
  - id: stable-uuid-seeder
    content: DefaultCategorySeeder 改为稳定 UUIDv5（命名空间+stableKey），并把系统预设写成 nameZh/nameEn 双语
    status: completed
  - id: ui-localized-display
    content: CategoryManager、Picker、Chart、Transaction row 等所有显示点改用 localizedName
    status: completed
  - id: soft-delete-ux
    content: CategoryManagerView 改为软删除、有子类时弹框批量确认、过滤 deletedAt!=nil
    status: completed
  - id: ck-category-schema
    content: LedgerShareService 添加 SharedCategory RecordType + CategoryField 枚举 + buildCategoryRecords(两趟 parentRef)
    status: completed
  - id: tx-schema-ref
    content: TxField.categoryPath 废弃，改为推 categoryRef (CKReference)；buildTransactionRecords 适配
    status: completed
  - id: push-category-everywhere
    content: prepareShare/addParticipant/pushSharedLedger 等所有 push 贯穿 Category；本地 CRUD 后触发防抖 push
    status: completed
  - id: import-refactor
    content: importRecords 重构：三桶 + Category 两趟 upsert + LWW(modificationDate vs updatedAt) + colorHex 首创写入
    status: completed
  - id: build-verify
    content: xcodebuild 编译通过，修复 lint 和新 API 引用
    status: completed
isProject: false
---

## 一、核心策略一锤定音

- **作用域**：Category 全局（所有 ledger 共用一张表），不引入 ledger 绑定
- **冲突合并**：last-write-wins，以 `CKRecord.modificationDate`（云端权威）为准，本地加 `updatedAt` 做 push 侧比对
- **删除**：软删除（`deletedAt` 字段）；禁止直接删父类，需先删光子类；被交易引用的分类删除后关系保留
- **权限**：`.readWrite` 参与者等同 owner，`.readOnly` 只拉不推（CloudKit ACL 天然保证，不加业务 gating）
- **系统预设**：改用命名空间稳定 UUID（A/B 两端种子即对齐），不做老数据迁移
- **i18n**：只支持中/英，Category 新增 `nameZh`/`nameEn` 两个字段同步；本地按 locale 选择显示
- **个性化字段不同步**：`isHidden`、`sortOrder`（完全本地）
- **颜色**：首次创建时同步 `colorHex`/`colorHexDark`，后续 update 不覆盖（允许个性化）
- **i 其他 ledger 污染、退出共享后保留所有同步过来的分类** —— 接受这些代价，不做清理逻辑

---

## 二、数据模型改动

### 2.1 `Category.swift`（`[Projects/FIREprint_App/ios_workspace/FIREprint/Models/Category.swift](Projects/FIREprint_App/ios_workspace/FIREprint/Models/Category.swift)`）

新增字段：

```swift
var updatedAt: Date = Date()        // 本地写权威时间
var deletedAt: Date?                // 软删除标记（nil = 存活）
var nameZh: String?                 // 中文名
var nameEn: String?                 // 英文名
```

新增计算属性：

```swift
var localizedName: String {
    let isChinese = Locale.current.language.languageCode?.identifier == "zh"
    return isChinese ? (nameZh ?? name) : (nameEn ?? name)
}
```

`fullPath` 改用 `localizedName` 拼接。所有 UI 点（CategoryManager / Picker / Charts / Transaction row）切到 `localizedName`。

所有本地写入 Category 的地方统一刷 `self.updatedAt = .now`（预计封装一个 `markDirty()` 方法避免漏刷）。

### 2.2 `DefaultCategorySeeder`

把系统预设改成**确定性 UUID + 双语名**。每个预设一个稳定 seed key：

```swift
struct PresetSpec {
    let stableKey: String   // e.g. "system.preset.food"
    let nameZh: String
    let nameEn: String
    let icon: String
    let colorHex: String
    let isPassive: Bool
    // ...
}

func stableUUID(for key: String) -> UUID {
    // 基于 DNS namespace 的 UUIDv5，跨设备结果一致
}
```

种子时 `Category.id = stableUUID(for: spec.stableKey)`。A、B 两端本地种出来的"餐饮"UUID 完全相同，后面云端同步天然对齐。

### 2.3 `Transaction` CloudKit schema 改动

`TxField.categoryPath` 字符串字段**废弃**，改为 `TxField.categoryRef`（CKReference 指向 SharedCategory record）。

---

## 三、CloudKit schema 与 `LedgerShareService` 改动

### 3.1 新增 RecordType：`SharedCategory`

在 `[Projects/FIREprint_App/ios_workspace/FIREprint/Infrastructure/LedgerShareService.swift](Projects/FIREprint_App/ios_workspace/FIREprint/Infrastructure/LedgerShareService.swift)`：

```swift
enum RecordType {
    static let ledger      = "SharedLedger"
    static let transaction = "SharedTransaction"
    static let category    = "SharedCategory"   // 新增
}

enum CategoryField {
    static let id         = "id"
    static let nameZh     = "nameZh"
    static let nameEn     = "nameEn"
    static let icon       = "icon"
    static let type       = "type"
    static let sortOrder  = "sortOrder"          // 可选：不同步也行，保留以后扩展
    static let colorHex   = "colorHex"
    static let colorHexDark = "colorHexDark"
    static let isPassive  = "isPassive"
    static let isSystemPreset = "isSystemPreset"
    static let parentRef  = "parent"             // CKReference -> SharedCategory (nullable)
    static let deletedAt  = "deletedAt"          // 软删除
    static let creatorAppleID = "creatorAppleID"
}
```

### 3.2 `buildCategoryRecords(_:)` 新增 builder

推送哪些 Category？策略：**全量推送用户所有（含系统预设，含 deletedAt 非空的）**。反正全局作用域。

```swift
private func buildCategoryRecords(context: ModelContext) -> [CKRecord] {
    // 1. fetch 所有 Category（包含已软删除）
    // 2. 第一趟：全量 build record（parentRef 暂不填）
    // 3. 第二趟：填 parentRef（CKReference.action = .none）
}
```

### 3.3 `buildTransactionRecords` 调整

去掉 `categoryPath`，改推 `categoryRef`：

```swift
if let cat = tx.category {
    let catRID = categoryRecordID(for: cat.id)
    record[TxField.categoryRef] = CKRecord.Reference(
        recordID: catRID, action: .none
    )
}
```

### 3.4 push 路径整合

`pushSharedLedger` / `prepareShare` / `addParticipant` 等凡是 push 交易的地方，都要**连带 push Category**。Category 推送应该在 Transaction 之前（同一批 modify 里 CloudKit 会按依赖顺序自己处理，但我们保证顺序更稳）。

### 3.5 `importRecords` 重构

改造成三桶 + 两趟 Category：

```swift
let ledgerRecords   = records.filter { $0.recordType == RecordType.ledger }
let categoryRecords = records.filter { $0.recordType == RecordType.category }
let txRecords       = records.filter { $0.recordType == RecordType.transaction }

upsertLedgers(ledgerRecords, into: context)

// Category 两趟
let catByUUID = upsertCategoryNodesPass1(categoryRecords, into: context)   // 不接 parent
linkCategoryParentsPass2(categoryRecords, catByUUID, into: context)

upsertTransactions(txRecords, catByUUID: catByUUID, into: context)

try context.save()
```

**upsert Category 的 LWW 逻辑**：

```swift
let remoteUpdatedAt = record.modificationDate ?? .distantPast
if let existing = existingCategory {
    if remoteUpdatedAt > existing.updatedAt {
        // 更新 nameZh/nameEn/icon/type/isPassive/isSystemPreset/deletedAt/parentRef
        // 但 colorHex/colorHexDark 仅在 existing 未设置时才写（允许个性化颜色）
        // 本地 isHidden/sortOrder 不动
        existing.updatedAt = remoteUpdatedAt
    }
} else {
    // 新建，colorHex 从 record 取
    let cat = Category(...)
    cat.updatedAt = remoteUpdatedAt
    context.insert(cat)
}
```

---

## 四、UX 和业务逻辑

### 4.1 `CategoryManagerView`（`[Projects/FIREprint_App/ios_workspace/FIREprint/Views/Settings/CategoryManagerView.swift](Projects/FIREprint_App/ios_workspace/FIREprint/Views/Settings/CategoryManagerView.swift)`）

- 列表过滤掉 `deletedAt != nil` 的分类
- 删除按钮逻辑改为：
  - 如果有非删除子类：弹框"该分类下还有 N 个子分类，是否一并删除？"→ 确认后对自己和所有子类一次性设 `deletedAt = .now`，`updatedAt = .now`
  - 无子类：直接软删
  - 交易引用不检查（按你的选择：软删后关系保留，不影响交易显示）
- 删除后触发一次 `pushSharedLedger` 的延伸：push 所有 Category（因为全局）

### 4.2 `ChartView` / 统计

所有显示 category 名字的地方用 `localizedName`。被软删除的分类如果还有历史交易引用，图表里仍然以 `localizedName` 显示（但不会出现在新建交易的 picker 里）。

### 4.3 `TransactionEditor` 的 Category Picker

过滤 `deletedAt == nil`，按 `localizedName` 显示。

---

## 五、同步触发点审计

确认以下路径都带上 Category push：
- `prepareShare(for:)` 首次分享
- `addParticipant(email:...)` 追加参与者
- `pushSharedLedger(_:)` / `pushAllOwnedSharedLedgers`
- 本地 Category CRUD 后需要触发 push（防抖，如 2s 合并一次）

import 侧：
- `acceptAndImport` 首次接受
- `pullAllSharedZones` / `pullOwnerSnapshot` 定时/launch 拉取
- 以上三者走同一个 `importRecords`，一次改造到位

---

## 六、验证项

编译后重点测这几条链路：
1. 两台全新 iPhone（A/B）分别装 app → seed 预设 → 校验两端"餐饮"UUID 一致（单测 `DefaultCategorySeeder` 的稳定 UUID）
2. A 创建自定义分类"宠物" → 共享 ledger 给 B → B 接受 → B 的 CategoryManager 出现"宠物"
3. A 改名"餐饮" → "吃喝" → B pull 后本地也变 → B 历史交易（哪怕是共享前的个人账本的）也显示"吃喝"
4. A 软删"旅行"（无子类无交易）→ B pull 后看不到
5. A 尝试删有子类的分类 → 弹框确认 → 批量软删 → B pull 后批量消失
6. B 自定义"猫粮"颜色改为紫色 → A push 的 Category update（仍然带它自己的颜色）不会覆盖 B 的紫色
7. 中英文设备切换 → 系统预设显示正确
8. readOnly 参与者尝试改分类 → CloudKit 拒绝写入（只要不崩）

每步完成后跑：

```bash
cd Projects/FIREprint_App/ios_workspace && xcodebuild -workspace FIREprint.xcworkspace -scheme FIREprint -destination 'generic/platform=iOS' -configuration Debug build 2>&1 | tail -30
```
