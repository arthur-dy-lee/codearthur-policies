---
name: Fastlane 使用说明
overview: 解释 Fastlane 在 SilenceCut 项目中的作用，说明当前配置状态、需要补充的内容，以及如何使用它来自动化 App Store 上架流程。
todos:
  - id: fill-team-id
    content: 在 Appfile 中填入 Apple Developer Team ID
    status: pending
  - id: write-fastfile
    content: 编写 Fastfile，定义 beta / release / metadata_only 等 lanes
    status: pending
  - id: create-gemfile
    content: 创建 Gemfile 锁定 fastlane 版本依赖
    status: pending
  - id: add-release-notes
    content: 为 13 个语言目录各创建 release_notes.txt
    status: pending
  - id: prepare-screenshots
    content: （可选）准备 App Store 截图或配置 snapshot 自动截图
    status: pending
isProject: false
---

# Fastlane 使用说明 -- SilenceCut 项目

## 一、Fastlane 是什么？有什么用？

Fastlane 是 iOS/Android 自动化部署工具，它能帮你**一条命令完成**以下事情：

- **自动构建** -- 编译打包 .ipa 文件
- **自动签名** -- 管理证书和 Provisioning Profile（通过 `match`）
- **自动截图** -- 批量在不同设备上生成 App Store 截图
- **上传 metadata** -- 把 name、description、keywords、screenshots 等信息批量上传到 App Store Connect
- **提交审核** -- 自动提交 App 到 Apple 审核
- **发布 TestFlight** -- 一键上传到 TestFlight 供测试

简单来说：**没有 Fastlane 你需要手动在 App Store Connect 网站上逐个语言填写信息、手动上传截图、手动打包提交。Fastlane 把这些全自动化了。**

## 二、当前项目状态

项目路径：`Projects/SilenceCut_App/ios_workspace/fastlane/`

### 已有内容（已完成）

- **[Appfile](Projects/SilenceCut_App/ios_workspace/fastlane/Appfile)** -- 基本配置（app_identifier、apple_id），但 `team_id` 为空
- **metadata/** -- 13 个语言的 App Store 文案**已全部填好**：
  - en-US, zh-Hans, zh-Hant, ja, ko, fr-FR, es-ES, pt-BR, ru, ar-SA, hi, id, vi
  - 每个语言包含：name, subtitle, description, keywords, promotional_text, privacy_url

### 缺失内容（需要补充）

| 文件 | 状态 | 说明 |
|------|------|------|
| `Fastfile` | 为空 | **最关键**，定义所有自动化操作（lane），没有它 Fastlane 什么都做不了 |
| `Appfile` 中 `team_id` | 为空 | 需要填入你的 Apple Developer Team ID |
| `Gemfile` | 不存在 | 锁定 Fastlane 版本，确保一致性 |
| `metadata/*/release_notes.txt` | 不存在 | 每个语言的"此版本新内容"（What's New），提交审核时需要 |
| `screenshots/` | 不存在 | App Store 截图，可以用 Fastlane 的 `snapshot` 自动生成，也可以手动放入 |

## 三、需要你填写 / 提供的信息

### 3.1 `team_id`（必填）

在 [Appfile](Projects/SilenceCut_App/ios_workspace/fastlane/Appfile) 中：

```
team_id ""  # <-- 填入你的 Apple Developer Team ID
```

查找方式：登录 https://developer.apple.com/account -> Membership Details -> Team ID（10位字母数字，如 `A1B2C3D4E5`）

### 3.2 `Fastfile`（必填 -- 核心文件）

[Fastfile](Projects/SilenceCut_App/ios_workspace/fastlane/Fastfile) 目前为空，需要定义以下 lanes：

- `beta` lane -- 构建并上传到 TestFlight
- `release` lane -- 构建、上传 metadata + 截图、提交审核
- `metadata_only` lane -- 仅上传 metadata（不重新构建）
- `screenshots` lane -- 自动截图（可选）

### 3.3 `release_notes.txt`（提交审核前必填）

每个语言目录下需要一个 `release_notes.txt`，内容是"此版本新内容"。例如：

```
en-US/release_notes.txt -> "Initial release. Remove video silence with AI."
zh-Hans/release_notes.txt -> "首次发布。AI 智能去除视频静音片段。"
```

## 四、如何使用

### 4.1 安装 Fastlane

```bash
# 方式一：Homebrew（推荐 macOS）
brew install fastlane

# 方式二：RubyGems
sudo gem install fastlane

# 方式三：Bundler（推荐团队协作）
cd Projects/SilenceCut_App/ios_workspace
bundle install
```

### 4.2 填好配置后的使用命令

```bash
cd Projects/SilenceCut_App/ios_workspace

# 仅上传/更新 App Store 文案（metadata）
fastlane metadata_only

# 构建并上传到 TestFlight
fastlane beta

# 构建、上传、提交 App Store 审核
fastlane release
```

### 4.3 典型工作流

```mermaid
flowchart LR
  A[填写metadata文案] --> B[fastlane beta]
  B --> C[TestFlight测试]
  C --> D[填写release_notes]
  D --> E[fastlane release]
  E --> F[App Store审核]
  F --> G[上架]
```

## 五、实施步骤

完成 Fastlane 配置需要按以下步骤进行：

1. **填写 `team_id`** -- 在 Appfile 中补充 Apple Developer Team ID
2. **编写 `Fastfile`** -- 定义 beta / release / metadata_only 等 lanes
3. **创建 `Gemfile`** -- 锁定 fastlane 版本
4. **添加 `release_notes.txt`** -- 为 13 个语言目录各添加一个版本说明文件
5. **准备截图**（可选） -- 放入 `fastlane/screenshots/` 或配置自动截图
