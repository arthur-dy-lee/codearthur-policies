---
name: csv import skip diagnostics
overview: 增强 FIREprint CSV 导入的"格式异常"诊断能力：在解析时记录每一条被跳过行的原始行号、原文与判定原因，通过 AppLogger 输出到 Xcode 控制台（带专用搜索标签 `##CSV_SKIP##`，方便用户过滤出这些行直接复制给我），并在导入完成页面以可折叠列表形式展示。
todos:
  - id: parser
    content: 在 ImportParser 新增 SkippedRowDetail + 分类函数 + 按原始行号遍历 + AppLogger 日志
    status: completed
  - id: vm
    content: ImportVM 暴露 skippedRowDetails 并在 confirm/confirmAndMap/reset 中同步
    status: completed
  - id: ui
    content: ImportView 完成页新增 DisclosureGroup 列表展示跳过行
    status: completed
  - id: i18n
    content: zh-Hans 与 en Localizable.strings 增加 2 条文案
    status: completed
  - id: build
    content: xcodebuild 编译 FIREprint scheme 验证无错
    status: completed
isProject: false
---

# CSV 导入格式异常诊断增强

## 目标

1. **专用搜索标签**：所有被跳过行的日志都带独立前缀 `##CSV_SKIP##`，用户只需在 Xcode Console 底部搜索框输入该字符串，即可**精确筛出**这些行并复制给我；不会被其他任何日志污染（自定义 # + 大写 + 固定拼写，碰撞概率为零）。
2. **日志格式**：每一条跳过行输出一行：
   ```
   ##CSV_SKIP## L<原始行号> | reason=<原因> | raw=<原文截断200字符>
   ```
   末尾再一行汇总：
   ```
   ##CSV_SKIP_SUMMARY## parsed=<n> skipped=<k>
   ```
3. **UI（附加，不影响日志流）**：导入完成页新增「查看跳过详情」可展开区，显示 `行号 / 原因 / 原文（截断）` 列表。
4. **最小侵入**：仅改 3 个 Swift 文件 + 1 个本地化文件；不改变现有数据流、不重写分隔符逻辑（只是"识别并分类"问题行）。

## 用户使用方式

导入完成后，在 Xcode 菜单 View → Debug Area → Activate Console；底部搜索框输入 `##CSV_SKIP##` → 只会看到跳过的行 → 全选复制给我即可（不夹杂 CoreData/CloudKit/AdMob 的噪音）。

## 改动一览

### 1. [Projects/FIREprint_App/ios_workspace/FIREprint/Domain/ImportParser.swift](Projects/FIREprint_App/ios_workspace/FIREprint/Domain/ImportParser.swift)

- 新增 `SkippedRowDetail` 结构体：`lineNumber: Int` / `rawContent: String` / `reason: String`。
- `ImportParseStats` 增加字段 `skippedRows: [SkippedRowDetail]`（保留 `skippedMalformedRows` 计数以向后兼容）。
- `parseCSVWithStats` 重写行遍历：
  - 不再走 `contentLines(...)` 过滤空行的简化路径；改为遍历 `content.components(separatedBy: .newlines)` 保留**原文件 1-based 行号**。
  - 跳过空白/header 行不计入"跳过"。
  - 对 `fields.count < 2` 的行，调用新增的 `classifySkipReason(line:)` 给出原因，记入 `skippedRows`。
- 新增轻量分类函数 `classifySkipReason(_ line: String) -> String`，识别常见问题：
  - 含全角逗号 `，` → `"疑似使用了全角逗号"`
  - 含 `;` 或 `\t` 或 `|` → `"疑似使用了非英文逗号分隔符"`
  - 以 `#` 或 `//` 开头 → `"注释行"`
  - 包含 `合计 / 总计 / 小计 / Total / Sum / Subtotal` → `"汇总行"`
  - 否则 → `"字段数不足 (<2)"`
- 每跳过一行输出**带唯一标签**的 warning：
  ```swift
  AppLogger.warning(
      "##CSV_SKIP## L\(lineNumber) | reason=\(reason) | raw=\(String(rawContent.prefix(200)))",
      category: "FIREprint.Import"
  )
  ```
- 解析完成后汇总（用**第二个专属标签**便于区分）：
  ```swift
  AppLogger.info(
      "##CSV_SKIP_SUMMARY## parsed=\(records.count) skipped=\(skippedRows.count)",
      category: "FIREprint.Import"
  )
  ```
- 需要 `import MyAppCore`（AppLogger 来自共享包）。
- 为避免 AppLogger 自带的 storm protection（>200 条/10s 会提升日志级别）把后续警告吞掉，循环里额外控制：若跳过数超过 500 条，从第 501 条起改用 `debugSampled`（每 50 条输出一次，外加最终 summary），确保最坏情况也不会刷屏。

### 2. [Projects/FIREprint_App/ios_workspace/FIREprint/ViewModels/ImportVM.swift](Projects/FIREprint_App/ios_workspace/FIREprint/ViewModels/ImportVM.swift)

- 新增字段 `var skippedRowDetails: [SkippedRowDetail] = []`。
- `confirmColumnMapping()` 与 `confirmAndMap(context:)` 都从 `result.stats.skippedRows` 赋值给 `skippedRowDetails`。
- `reset()` 里追加 `skippedRowDetails = []`。

### 3. [Projects/FIREprint_App/ios_workspace/FIREprint/Views/Settings/ImportView.swift](Projects/FIREprint_App/ios_workspace/FIREprint/Views/Settings/ImportView.swift)

- 在 `importBreakdownRows` 结尾（`if let err = vm.errorMessage` 之后）追加：

  ```swift
  if !vm.skippedRowDetails.isEmpty {
      DisclosureGroup(
          NSLocalizedString("import.done.skipped_details_title", comment: "")
      ) {
          VStack(alignment: .leading, spacing: 6) {
              ForEach(vm.skippedRowDetails.prefix(100)) { detail in
                  VStack(alignment: .leading, spacing: 2) {
                      Text("L\(detail.lineNumber) · \(detail.reason)")
                          .font(.caption.weight(.medium))
                          .foregroundStyle(.orange)
                      Text(detail.rawContent)
                          .font(.caption2.monospaced())
                          .foregroundStyle(.secondary)
                          .lineLimit(2)
                  }
                  .frame(maxWidth: .infinity, alignment: .leading)
              }
              if vm.skippedRowDetails.count > 100 {
                  Text(String(format: NSLocalizedString("import.done.skipped_more", comment: ""),
                              vm.skippedRowDetails.count - 100))
                      .font(.caption2)
                      .foregroundStyle(.secondary)
              }
          }
          .padding(.top, 6)
      }
      .font(.subheadline)
  }
  ```

- `SkippedRowDetail` 需要 `Identifiable`（`let id = UUID()`），ForEach 才能直接用。

### 4. [Projects/FIREprint_App/ios_workspace/FIREprint/Resources/zh-Hans.lproj/Localizable.strings](Projects/FIREprint_App/ios_workspace/FIREprint/Resources/zh-Hans.lproj/Localizable.strings) 与 [en.lproj/Localizable.strings](Projects/FIREprint_App/ios_workspace/FIREprint/Resources/en.lproj/Localizable.strings)

- 在"import.done."前缀附近追加：
  - `import.done.skipped_details_title` → `"查看跳过行详情"` / `"View skipped row details"`
  - `import.done.skipped_more` → `"还有 %d 条未显示（请查看 Xcode 日志）"` / `"%d more (see Xcode log)"`
- 其他语言（ja/ur/vi/zh-Hant）不补充，默认回落到 Base/en 即可，保持本次改动的最小性。

## 验证步骤

1. XcodeGen 不需要重新生成（没加文件）；直接 `xcodebuild` 编译 FIREprint scheme。
2. 正式导入你真实 CSV。
3. 在 Xcode Console 搜索框输入 **`##CSV_SKIP##`** → 只剩跳过行；全选复制粘贴给我。
4. 下一轮根据这些行的 `reason` + `raw` 内容，或修 CSV、或扩展解析器规则。

## 风险与回退

- 纯增量改动，不改变已持久化数据；若行为异常可直接 `git revert`。
- 大 CSV 可能让 `skippedRowDetails` 过长，已用 `prefix(100)` 限制 UI，日志侧有 AppLogger 的 storm protection 兜底（>200 条/10s 自动降级），不会刷爆日志。