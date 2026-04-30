---
name: fastlane beta + 自动截图
overview: 给 FIREprint 加两个 fastlane 能力：(1) beta lane 自动构建并上传 TestFlight（不提审、不分发外部），(2) 全自动本地截图（先只跑 en-US，未来加语言只改 Snapfile 一行）。分两阶段实施，阶段 1 不依赖阶段 2。
todos:
  - id: phase1_gitignore
    content: 新建 fastlane/.gitignore 和 asc_api_key.sample.json，防止 .p8 和 api key json 进 git
    status: completed
  - id: phase1_appfile
    content: 等用户提供 team_id 后，补全 fastlane/Appfile
    status: completed
  - id: phase1_fastfile_beta
    content: 在 Fastfile 里新增 beta lane：increment_build_number + gym + pilot，显式 skip_submission + distribute_external=false
    status: completed
  - id: phase2_uitest_target
    content: 在 project.yml 中添加 FIREprintUITests target，类型为 bundle.ui-testing，挂在 FIREprint scheme 的 testAction 上
    status: completed
  - id: phase2_snapshot_helper
    content: 创建 FIREprintUITests/ 目录，放入 SnapshotHelper.swift（fastlane 官方）和 FIREprintSnapshotTests.swift（首页/Transactions/FIRE/Settings 截图脚本）
    status: completed
  - id: phase2_seed_data
    content: 在 FIREprint 主工程加 UITestSeed 钩子：看到 -uitest-screenshot launch argument 就跳过引导/ATT/UMP 并火入 ~10 条样本交易，避免空状态截图
    status: completed
  - id: phase2_snapfile
    content: 新建 fastlane/Snapfile：devices 列 iPhone 16 Pro Max / iPhone 15 / iPad Pro 13，languages 只列 en-US（留注释示例扩展）
    status: completed
  - id: phase2_fastfile_screenshots
    content: 在 Fastfile 新增 screenshots 和 upload_screenshots 两条 lane
    status: completed
  - id: phase1_test_run
    content: xcodegen generate + xcodebuild 编译验证；指导用户跑 fastlane beta 做端到端验证（依赖 ASC API Key）
    status: completed
  - id: phase2_test_run
    content: 跑 fastlane screenshots 验证全设备 en-US 截图产出正常
    status: completed
  - id: commit_all
    content: "分两个 commit 提交：feat(fastlane): add beta lane for TestFlight upload / feat(fastlane): add automated en-US screenshots via snapshot"
    status: in_progress
isProject: false
---

## 目标回顾

- 能做到 1：截图自动化，先跑 en-US，未来加语言改一行
- 能做到 2：只自动发 TestFlight，不自动提交审核（release 保持手动）
- 鉴权：App Store Connect API Key（.p8）

## 总体结构

```mermaid
flowchart LR
    dev["本地: fastlane beta"] --> bump["increment_build_number"]
    bump --> gym["gym 构建 Release ipa"]
    gym --> pilot["pilot 上传 TestFlight"]
    pilot --> done["ASC 处理中<br/>不提审 不分发"]

    dev2["本地: fastlane screenshots"] --> snapshot["snapshot 跑 UITest"]
    snapshot --> collect["screenshots/en-US/*.png"]
    collect --> deliver["upload_screenshots: deliver 上传"]
```

两条 lane 完全独立，可以分阶段做。

---

## 阶段 1: TestFlight beta lane（优先，阻塞小）

### 1.1 ASC API Key 准备（你手工做，一次性）

步骤（fastlane 不能替代）：

1. 登录 `https://appstoreconnect.apple.com` → **Users and Access** → **Integrations** → **App Store Connect API** → **Team Keys**
2. 点 **Generate API Key**
   - Name: `FIREprint Fastlane`
   - Access: **App Manager**（最低权限，够用）
3. 下载 `AuthKey_XXXXXXXXXX.p8`（只能下载一次！丢了只能重生成）
4. 记下：**Key ID**（10 位字符）、**Issuer ID**（UUID，页面顶部）

### 1.2 文件落位

- `.p8` 文件放到 `Projects/FIREprint_App/ios_workspace/fastlane/AuthKey_XXXXXXXXXX.p8`
- 新建 [Projects/FIREprint_App/ios_workspace/fastlane/asc_api_key.json](Projects/FIREprint_App/ios_workspace/fastlane/asc_api_key.json)：
  ```json
  {
    "key_id": "XXXXXXXXXX",
    "issuer_id": "abcd1234-...-...",
    "key": "fastlane/AuthKey_XXXXXXXXXX.p8",
    "duration": 1200,
    "in_house": false
  }
  ```
- 把 `.p8` 和 `asc_api_key.json` 加到 [Projects/FIREprint_App/ios_workspace/fastlane/.gitignore](Projects/FIREprint_App/ios_workspace/fastlane/.gitignore)（**绝不进 git**）
- 提交一份 `asc_api_key.sample.json` 给后来者参考

### 1.3 修 [Projects/FIREprint_App/ios_workspace/fastlane/Appfile](Projects/FIREprint_App/ios_workspace/fastlane/Appfile)

补 `team_id`（现在是空的）：

```ruby
app_identifier "com.codearthur.matrixapps.fireprint"
apple_id "paincupid@gmail.com"
team_id "XXXXXXXXXX"  # 从 ASC 页面右上角 Membership 找
```

### 1.4 改写 [Projects/FIREprint_App/ios_workspace/fastlane/Fastfile](Projects/FIREprint_App/ios_workspace/fastlane/Fastfile)

保留已有 `upload_metadata` lane，新增：

```ruby
default_platform(:ios)

API_KEY_PATH = "fastlane/asc_api_key.json"

platform :ios do

  desc "Upload only App Store metadata"
  lane :upload_metadata do
    deliver(
      api_key_path: API_KEY_PATH,
      skip_binary_upload: true,
      skip_screenshots: true,
      force: true,
      precheck_include_in_app_purchases: false
    )
  end

  desc "Build and upload to TestFlight (internal only, no review, no external distribution)"
  lane :beta do
    next_build = latest_testflight_build_number(
      api_key_path: API_KEY_PATH,
      initial_build_number: 0
    ) + 1

    increment_build_number(
      build_number: next_build,
      xcodeproj: "FIREprint.xcodeproj"
    )

    gym(
      project: "FIREprint.xcodeproj",
      scheme: "FIREprint",
      configuration: "Release",
      export_method: "app-store",
      output_directory: "./build",
      output_name: "FIREprint.ipa",
      clean: true,
      include_bitcode: false
    )

    pilot(
      api_key_path: API_KEY_PATH,
      ipa: "./build/FIREprint.ipa",
      skip_waiting_for_build_processing: true,
      skip_submission: true,
      distribute_external: false,
      changelog: ENV["FL_CHANGELOG"] || "Internal test build"
    )
  end
end
```

**关键参数，兑现"不 release"承诺**：
- `skip_submission: true` → 不提交 App Store 审核
- `distribute_external: false` → 不分发给外部测试组
- `skip_waiting_for_build_processing: true` → 上传完就退出（ASC 处理 ~20 分钟后你自己点"添加到测试组"）

### 1.5 使用方式

```bash
cd Projects/FIREprint_App/ios_workspace
bundle exec fastlane beta
# 或自定义 changelog
FL_CHANGELOG="fix ATT cold start" bundle exec fastlane beta
```

build number 自动从 ASC 最新 +1，不会撞号。MARKETING_VERSION 仍从 `project.yml` 改（手动）。

---

## 阶段 2: 截图自动化（先 en-US，后续可扩展）

### 2.1 在 [Projects/FIREprint_App/ios_workspace/project.yml](Projects/FIREprint_App/ios_workspace/project.yml) 加 UITest target

在现有 `FIREprint` target 后面加：

```yaml
  FIREprintUITests:
    type: bundle.ui-testing
    platform: iOS
    sources:
      - path: FIREprintUITests
    dependencies:
      - target: FIREprint
    settings:
      base:
        SWIFT_VERSION: "5.9"
        PRODUCT_BUNDLE_IDENTIFIER: com.codearthur.matrixapps.fireprint.uitests
        TEST_TARGET_NAME: FIREprint
```

然后在 schemes 里给 FIREprint scheme 加 testAction 挂这个 target（xcodegen 自动处理）。

### 2.2 新建目录与文件

- [Projects/FIREprint_App/ios_workspace/FIREprintUITests/SnapshotHelper.swift](Projects/FIREprint_App/ios_workspace/FIREprintUITests/SnapshotHelper.swift) — fastlane 官方 helper（`fastlane snapshot init` 生成，或从 SilenceCut 复制若有）
- [Projects/FIREprint_App/ios_workspace/FIREprintUITests/FIREprintSnapshotTests.swift](Projects/FIREprint_App/ios_workspace/FIREprintUITests/FIREprintSnapshotTests.swift) — 脚本：

  ```swift
  import XCTest

  final class FIREprintSnapshotTests: XCTestCase {
      override func setUpWithError() throws {
          continueAfterFailure = false
          let app = XCUIApplication()
          setupSnapshot(app)
          app.launchArguments += ["-uitest-screenshot"]
          app.launch()
      }

      func testTakeScreenshots() throws {
          let app = XCUIApplication()
          snapshot("01_Dashboard")

          app.tabBars.buttons.element(boundBy: 1).tap()  // Transactions
          snapshot("02_Transactions")

          app.tabBars.buttons.element(boundBy: 2).tap()  // FIRE progress
          snapshot("03_FIRE")

          app.tabBars.buttons.element(boundBy: 3).tap()  // Settings
          snapshot("04_Settings")
      }
  }
  ```

### 2.3 App 侧增加 seed sample data 钩子

新建 [Projects/FIREprint_App/ios_workspace/FIREprint/Infrastructure/UITestSeed.swift](Projects/FIREprint_App/ios_workspace/FIREprint/Infrastructure/UITestSeed.swift)：

```swift
import Foundation

enum UITestSeed {
    static var isScreenshotMode: Bool {
        ProcessInfo.processInfo.arguments.contains("-uitest-screenshot")
    }
}
```

在 [Projects/FIREprint_App/ios_workspace/FIREprint/App/FIREprintApp.swift](Projects/FIREprint_App/ios_workspace/FIREprint/App/FIREprintApp.swift) 的 `init()` 或 `ContentView.onAppear` 里：若 `UITestSeed.isScreenshotMode` 为真，跳过引导、跳过 ATT/UMP、灌入 ~10 条样本交易。避免空状态截图。

### 2.4 新建 [Projects/FIREprint_App/ios_workspace/fastlane/Snapfile](Projects/FIREprint_App/ios_workspace/fastlane/Snapfile)

```ruby
devices([
  "iPhone 16 Pro Max",
  "iPhone 15",
  "iPad Pro 13-inch (M4)"
])

languages([
  "en-US"
  # 未来加多语言：
  # "zh-Hans",
  # "zh-Hant",
  # "ja"
])

scheme("FIREprint")
output_directory("./fastlane/screenshots")
clear_previous_screenshots(true)
override_status_bar(true)
stop_after_first_error(false)
concurrent_simulators(true)
```

### 2.5 Fastfile 增加两条 lane

```ruby
  desc "Capture localized screenshots via UITests (only languages listed in Snapfile)"
  lane :screenshots do
    capture_ios_screenshots
  end

  desc "Upload screenshots to App Store Connect (no binary, no metadata)"
  lane :upload_screenshots do
    deliver(
      api_key_path: API_KEY_PATH,
      skip_binary_upload: true,
      skip_metadata: true,
      overwrite_screenshots: true,
      force: true
    )
  end
```

### 2.6 使用方式

```bash
bundle exec fastlane screenshots        # 本地跑模拟器截图
bundle exec fastlane upload_screenshots  # 上传已存在的截图
```

加语言：只改 `Snapfile` 的 `languages` 数组 + 在 `metadata/<locale>/` 里补对应 description（已经全 13 语言目录就绪）。

---

## 阶段边界

| 阶段 | 产出 | 你需要手工做的 | 预计工时 |
|---|---|---|---|
| 1. beta lane | 能一行命令上传 TestFlight | 生成 ASC API Key、提供 Key ID / Issuer ID / team_id | 15 min |
| 2. 截图自动化 | 能一行命令跑出 en-US 全设备截图 | 跑一次检查导航路径、必要时调脚本 | 1~2 h |

---

## 需要你提供的信息

| 字段 | 去哪拿 |
|---|---|
| ASC API Key ID | ASC → Users and Access → Integrations → API Keys |
| ASC Issuer ID | 同上页面顶部 |
| `.p8` 文件 | 同上点 Generate 后下载 |
| team_id（10 位） | ASC 右上角账号菜单 → Membership |

拿到后我执行阶段 1；阶段 2 可以立刻并行做（不依赖 API Key，只跑本地模拟器）。

## 不做的事

- 不引入 `match`（单人开发用 Xcode 自动签名足够）
- 不加 `appicon` lane（现在图标不频繁改）
- 不自动提交审核、不自动分发外部测试组（按你的要求"release 手动点"）
- 不改 metadata（已在另一个 commit 里搞定）