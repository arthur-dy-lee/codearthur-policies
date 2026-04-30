---
name: fastlane setup fireprint
overview: 在 Projects/FireUp_App/ios_workspace/ 下新建与 SilenceCut 完全同构的 fastlane 目录：Appfile + 预置 upload_metadata 单 lane 的 Fastfile + 13 个 App Store Connect 支持的 locale，每 locale 下 6 个 txt 占位文件，privacy_url 按 codearthur-policies 规则预先填好。
todos:
  - id: mkdirs
    content: 创建 fastlane/ 主目录与 13 个 metadata/<locale>/ 子目录
    status: completed
  - id: appfile
    content: 写入 Appfile（app_identifier / apple_id / team_id）
    status: completed
  - id: fastfile
    content: 写入 Fastfile，只包含 upload_metadata 单 lane
    status: completed
  - id: metadata_privacy_url
    content: 13 个 locale 写入统一的 privacy_url.txt（指向 codearthur-policies/bozhu/en/privacy.html?name=FIREprint）
    status: completed
  - id: metadata_others
    content: 13 个 locale 写入 name.txt（FIREprint）+ 4 个 TODO 占位 txt
    status: completed
  - id: verify
    content: 运行 fastlane lanes 验证 Fastfile/Appfile 语法正确
    status: completed
  - id: commit
    content: "按 conventional commit 提交：feat: scaffold fastlane for FIREprint with 13-locale metadata placeholders"
    status: completed
isProject: false
---

## 背景与约束

- 本机已全局装好 `fastlane 2.232.2`（Homebrew），不使用 `Gemfile` / `gem install`，避免与 Homebrew ruby/fastlane 冲突。
- 目标目录：[Projects/FireUp_App/ios_workspace/fastlane/](Projects/FireUp_App/ios_workspace/fastlane/)（与 SilenceCut 同层，不要放到 `FIREprint.xcodeproj/` 内部）。
- Bundle ID：`com.codearthur.matrixapps.fireprint`（来自 [Projects/FireUp_App/ios_workspace/project.yml](Projects/FireUp_App/ios_workspace/project.yml)，不改动）。
- Apple ID：`paincupid@gmail.com`（与 SilenceCut 一致）；`team_id` 留空（与 SilenceCut 一致，通过 Xcode signing 自动解析；需要时再填 `XVZHPD648U`）。
- App Store 展示名占位用 `FIREprint`（后续由你精修；注意 App Store 名字上限 30 字符、副标题 30、关键词 100、promotional_text 170、description 4000）。

## 产物结构

```
Projects/FireUp_App/ios_workspace/fastlane/
├── Appfile
├── Fastfile
└── metadata/
    ├── ar-SA/  en-US/  es-ES/  fr-FR/  hi/  id/  ja/  ko/  pt-BR/  ru/  vi/  zh-Hans/  zh-Hant/
    └── 每个 locale 下 6 个文件：
        name.txt  subtitle.txt  keywords.txt  description.txt  promotional_text.txt  privacy_url.txt
```

**locale 选择说明**：FIREprint 的 `project.yml` 声明了 15 个 knownRegions，其中 `ur` 和 `bn` **不在 App Store Connect 支持列表中**，因此 fastlane metadata 按 SilenceCut 完全相同的 13 个 locale 建立（App 内本地化 strings 仍可保留 15 种，二者互不影响）。

## 关键文件内容

### `fastlane/Appfile`（参照 SilenceCut 写法）

```ruby
app_identifier "com.codearthur.matrixapps.fireprint"
apple_id "paincupid@gmail.com"
team_id ""
```

### `fastlane/Fastfile`（minimal：只预置 `upload_metadata` 一个 lane）

```ruby
default_platform(:ios)

platform :ios do
  desc "Upload only App Store metadata (no binary, no screenshots)"
  lane :upload_metadata do
    deliver(
      skip_binary_upload: true,
      skip_screenshots: true,
      force: true,
      precheck_include_in_app_purchases: false
    )
  end
end
```

用法：`cd Projects/FireUp_App/ios_workspace && fastlane upload_metadata`

未来需要截图上传 / TestFlight / 正式提交审核时，再向该 Fastfile 追加 `upload_screenshots` / `beta` / `release` lane（保持与 SilenceCut 同步演进）。

### `metadata/<locale>/privacy_url.txt`（全部 13 个 locale 统一写同一行，与 SilenceCut 风格一致）

```
https://arthur-dy-lee.github.io/codearthur-policies/bozhu/en/privacy.html?name=FIREprint
```

注意：App Store Connect 的 metadata 字段**只有 `privacy_url`**，没有 `terms_url` 或 `account_deletion_url`。条款与账号删除链接按 [codearthur-policies/README.md](/Users/arthur.lee/codes/codearthur-policies/README.md) 的 Swift 示例集成到 App 设置页即可，与 fastlane 无关。

### 其余 5 个 txt 文件的 placeholder 策略

- `name.txt`：写 `FIREprint`（13 个 locale 统一，待你按地区本地化精修）
- `subtitle.txt`：写 `# TODO: subtitle <=30 chars`
- `keywords.txt`：写 `# TODO: comma,separated,<=100chars`
- `description.txt`：写 `# TODO: description <=4000 chars`
- `promotional_text.txt`：写 `# TODO: promotional text <=170 chars`

（`#` 前缀是占位标记而非 fastlane 注释语法——fastlane metadata txt 没有注释机制；首次 `upload_metadata` 前你需要清理掉 TODO 内容，否则会把 TODO 文案真的推到 App Store Connect。计划中会在 Fastfile 添加一条 `UI.user_error!` 的前置保护并不激进，我倾向保持与 SilenceCut 简洁风格一致，**不加保护**，由你人工 review。如果你希望加保护可告诉我。）

## 执行步骤

1. 新建目录：`Projects/FireUp_App/ios_workspace/fastlane/` 与 13 个 `metadata/<locale>/` 子目录
2. 写入 `Appfile`（3 行）
3. 写入 `Fastfile`（minimal 版 `upload_metadata` lane）
4. 在每个 locale 目录下创建 6 个 txt（`name` + `privacy_url` 为真实内容，其余 4 个为 TODO 占位）
5. 本地验证：`cd Projects/FireUp_App/ios_workspace && fastlane lanes`（应列出 `upload_metadata`），然后 `fastlane upload_metadata --verbose` 在未登录 ASC 的情况下会提示身份验证，此步不实际推送（仅验证 Fastfile/Appfile 语法）
6. **不涉及 Swift 源码改动，不需要 xcodebuild 编译验证**
7. 完成后按 CLAUDE.md auto-commit 规范提交：`feat: scaffold fastlane for FIREprint with 13-locale metadata placeholders`

## 不做的事情（明确排除）

- 不生成 `Gemfile` / `Gemfile.lock`（按你指示走 Homebrew 全局 fastlane）
- 不在 FIREprint 项目中嵌入 `PolicyURLs` Swift 代码（README 中的 Swift 集成属于 App 内设置页工作，不在本次 fastlane scaffold 范围；如需可后续单独做）
- 不上传任何截图（screenshots 目录本次不建，与 SilenceCut 当前状态一致）
- 不触碰 `project.yml` / `xcodegen` / 签名配置