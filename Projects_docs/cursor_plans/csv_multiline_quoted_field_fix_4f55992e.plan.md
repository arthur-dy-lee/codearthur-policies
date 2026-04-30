---
name: csv multiline quoted field fix
overview: 修复 FIREprint CSV 解析器对"引用字段内含换行"（RFC 4180 多行字段）的错误切分：目前按物理 \n 先切记录再切字段，导致长备注字段里的 19 行装修明细被误当作独立 CSV 行处理（1 条被截断 + 18 条伪跳过）。改为按引号状态切记录后再切字段，并同步支持 `""` 双引号转义。
todos:
  - id: split
    content: ImportParser 新增 splitIntoRecords：按引号状态切记录，支持 \r\n / "" 转义
    status: completed
  - id: parse-csv
    content: parseCSVWithStats 切换到 splitIntoRecords 的记录列表，其余逻辑不动
    status: completed
  - id: parse-line
    content: parseCSVLine 支持 "" 字面量双引号转义
    status: completed
  - id: header
    content: parseHeader 同步切换到 splitIntoRecords 管道
    status: completed
  - id: build
    content: xcodebuild 编译 FIREprint 验证无编译错误
    status: completed
isProject: false
---

# CSV 多行引用字段解析修复

## 问题根因

[ImportParser.swift 第 184-188 行](Projects/FIREprint_App/ios_workspace/FIREprint/Domain/ImportParser.swift) 处理顺序错：

```
content.components(separatedBy: .newlines)   ← 还不知道引号状态就按 \n 切
  → 每"行" parseCSVLine                         ← 为时已晚
```

RFC 4180 规定：`"..."` 引用字段里的 `\n` 属于字段内容，不是记录分隔符。Excel 遵守这条规则，我们的解析器不遵守。

## 当前产生的错误行为（L2599 那条装修流水）

```mermaid
flowchart TD
    Raw["CSV 原文 (跨 20 物理行)<br/>支出,...,34922.00,...,\"拆墙 700 TAB 700<br/>床3个,沙发...<br/>...<br/>TAB 34922\""]
    Raw --> Split["components(separatedBy: .newlines)<br/>切出 20 条'行'"]
    Split --> First["L2599: 支出,...,34922.00,...,\"拆墙 700 TAB 700<br/>(引号未闭合但已到行尾)"]
    Split --> Mid["L2600-L2632: 19 条独立'行'<br/>(本该是 L2599 的字段内容)"]
    First --> Parsed["parseCSVLine 返回 11 字段<br/>末字段=拆墙 700 TAB 700<br/>金额 34922 导入正确<br/>但备注截断"]
    Mid --> Skipped["18 条被 CSV_SKIP<br/>(字段数不足 / TAB分隔 / 全角逗号)"]
```

## 解决方案（单点修复）

仅改一个文件：[Projects/FIREprint_App/ios_workspace/FIREprint/Domain/ImportParser.swift](Projects/FIREprint_App/ios_workspace/FIREprint/Domain/ImportParser.swift)。

### 改动 1：新增 `splitIntoRecords`（替换按 `\n` 切分的逻辑）

按**引号状态**切记录，返回原始起始行号 + 完整记录文本（可能跨多行）。

```swift
private static func splitIntoRecords(_ content: String) -> [(startLine: Int, text: String)] {
    var records: [(Int, String)] = []
    var current = ""
    var inQuotes = false
    var recordStartLine = 1
    var currentLine = 1
    var i = content.startIndex

    while i < content.endIndex {
        let ch = content[i]
        if ch == "\"" {
            let next = content.index(after: i)
            if inQuotes, next < content.endIndex, content[next] == "\"" {
                // RFC 4180: "" 在引用字段内表示字面量 "
                current.append("\"")
                current.append("\"")
                i = content.index(after: next)
                continue
            }
            inQuotes.toggle()
            current.append(ch)
        } else if ch == "\r" {
            // 统一忽略 \r（\r\n 的 \r 或者裸 \r），真正的换行语义交给 \n
            if !inQuotes {
                // do nothing: \n 会处理记录边界
            } else {
                current.append(ch)
            }
        } else if ch == "\n" {
            if inQuotes {
                current.append(ch)
                currentLine += 1
            } else {
                if !current.trimmingCharacters(in: .whitespaces).isEmpty {
                    records.append((recordStartLine, current))
                }
                current = ""
                currentLine += 1
                recordStartLine = currentLine
            }
        } else {
            current.append(ch)
        }
        i = content.index(after: i)
    }
    if !current.trimmingCharacters(in: .whitespaces).isEmpty {
        records.append((recordStartLine, current))
    }
    return records
}
```

### 改动 2：`parseCSVWithStats` 切换到记录级循环

把当前的：
```swift
let rawLines = content.components(separatedBy: .newlines)
var indexed: [(originalLine: Int, text: String)] = []
for (i, line) in rawLines.enumerated() { ... }
```
替换为：
```swift
let indexed = splitIntoRecords(content)
    .map { (originalLine: $0.startLine, text: $0.text) }
```

其余逻辑（`findHeaderIndex`、列映射构造、`parseCSVLine`、`classifySkipReason`、`##CSV_SKIP##` 日志、`##CSV_SKIP_SUMMARY##`）**完全不动**。

### 改动 3：`parseCSVLine` 支持 `""` 字面量转义

当前代码（第 279-295 行）：
```swift
if char == "\"" {
    inQuotes.toggle()
} else if ...
```
改成"向前看一字符"判断是否为转义 `""`：遍历改成索引遍历，遇到 `"` 时若紧跟另一个 `"` 且处于 `inQuotes`，当作字面量追加到 `current`；否则 toggle。这确保跨多行的引用字段里合法出现的双引号（如 `"他说""好"` → `他说"好`）不会导致字段边界错乱。

### 改动 4：`parseHeader` 也走新管道

当前 `parseHeader` 调用 `contentLines(...)`。为一致性改成：
```swift
let records = splitIntoRecords(content)
guard !records.isEmpty else { throw ImportError.emptyFile }
let headerIdx = findHeaderIndex(lines: records.map(\.text))
return parseCSVLine(records[headerIdx].text).map { $0.trimmingCharacters(in: .whitespaces) }
```
保证"选列映射"和"实际解析"看到的记录列表完全一致（避免header index 错位）。

## 预期效果

| 指标 | 改前 | 改后 |
|---|---|---|
| `##CSV_SKIP_SUMMARY##` | `parsed=6811 skipped=18` | `parsed=6811 skipped=0` |
| L2599 装修流水备注 | 截断到 `拆墙 700\t700` | 完整 20 行明细保留 |
| 金额 34922 | 已正确 | 仍正确 |
| 其他 6810 行 | 正常 | 正常（相同代码路径） |
| `##CSV_SKIP##` 标签 | 保留 | 保留（以后真有坏行仍会告警） |

## 不改的部分

- `ImportVM`、`ImportView`、`Localizable.strings` 不动
- `SkippedRowDetail` / DisclosureGroup / `classifySkipReason` 保留（未来真损坏的行仍需要它们）
- `##CSV_SKIP##` / `##CSV_SKIP_SUMMARY##` 日志格式不变

## 验证步骤

1. `xcodebuild -project FIREprint.xcodeproj -scheme FIREprint -destination 'generic/platform=iOS' -configuration Debug build` 确保编译通过。
2. 用户重新安装到真机 → 再次导入同一份 CSV。
3. 在 Xcode Console 搜索 `##CSV_SKIP##`：应该看不到任何行（如果还有，把那些行贴过来，属于其他未知问题）。
4. 搜索 `##CSV_SKIP_SUMMARY##`：`parsed=N skipped=0`。
5. 在 App 里找到 2023-08-22 22:39:39 那条装修流水，查看备注 → 应该能看到完整 20 行明细。

## 风险 & 回退

- 纯解析层变更，不动数据库 schema 和 UI。
- 极端情况：CSV 里有裸 `"`（既不是字段开头也不是转义）的文件 → 可能产生解析错位；但当前代码同样会错，没有劣化。
- 一键回退：`git revert` 本次 commit。