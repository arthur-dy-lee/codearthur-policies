---
name: fastlane screenshots scaffold
overview: 在 FIREprint 的 fastlane 目录下新增 `screenshots/en-US/` 空目录与一份 README，确立"主语言英文截图 + 其他 12 locale 由 App Store fallback"的截图策略。
todos:
  - id: mkdir
    content: 创建 screenshots/en-US/ 目录
    status: pending
  - id: gitkeep
    content: 写入 screenshots/en-US/.gitkeep 空文件
    status: pending
  - id: readme
    content: 写入 screenshots/README.md（截图规范与工作流说明）
    status: pending
  - id: commit
    content: 仅添加这两个新文件并提交，不牵涉目录改名 diff
    status: completed
isProject: false
---

## 前提确认

- 目录已从 `FireUp_App` 改名为 `FIREprint_App`，新路径：[Projects/FIREprint_App/ios_workspace/fastlane/](Projects/FIREprint_App/ios_workspace/fastlane/)
- 既有 fastlane 结构（Appfile / Fastfile / 13 个 metadata locale）已随目录改名出现在新路径下，**本次不改动**
- 仅 iPhone（`TARGETED_DEVICE_FAMILY: "1"`），不需要 iPad 截图

## 产物

```
Projects/FIREprint_App/ios_workspace/fastlane/
└── screenshots/
    ├── README.md        ← 截图规范与工作流说明
    └── en-US/
        └── .gitkeep     ← 占位使 git 能跟踪空目录
```

**不创建** `screenshots/zh-Hans/` / `zh-Hant/` 或其他 locale 目录。其他 12 个 locale 的用户在 App Store 上会**自动看到 en-US 的截图**（Apple 原生 fallback 行为），完全合规。

## 文件内容

### `screenshots/en-US/.gitkeep`

空文件，仅用于让 git 跟踪该目录。添加真实截图后可删除（也可保留无影响）。

### `screenshots/README.md`

```markdown
# FIREprint App Store Screenshots

## Policy
- Only `en-US/` screenshots are maintained. All other locales fall back to
  `en-US` automatically via App Store Connect's built-in fallback. This is
  fully compliant with Apple App Store review rules.
- To add localized screenshots later (e.g. `zh-Hans/`), simply create a
  sibling folder with the same filename structure and re-run
  `fastlane upload_metadata` (or a future `upload_screenshots` lane).

## Device Matrix (iPhone only)
FIREprint targets iPhone only (TARGETED_DEVICE_FAMILY = 1). Prepare one set
of screenshots at the largest supported size; Apple will scale down for
smaller display classes.

- Primary: iPhone 6.9" (1320 x 2868) — iPhone 16 Pro Max
- Fallback: iPhone 6.7" (1290 x 2796) — iPhone 14/15 Pro Max

## File Naming
Place 5-10 PNG files in `en-US/`, named with a leading order prefix:

```
01_hero.png
02_feature_tracking.png
03_feature_chart.png
...
```

Fastlane uploads in filename sort order; the first file becomes the
featured screenshot on the App Store listing.

## Upload
```
cd Projects/FIREprint_App/ios_workspace
fastlane upload_metadata     # current lane also uploads screenshots if present
```

Note: the current `upload_metadata` lane sets `skip_screenshots: true`.
To actually push screenshots, either (a) temporarily flip the flag to
`false`, or (b) add a dedicated `upload_screenshots` lane later.
```

## 执行步骤

1. 创建目录 `Projects/FIREprint_App/ios_workspace/fastlane/screenshots/en-US/`
2. 写入 `screenshots/en-US/.gitkeep`（0 字节）
3. 写入 `screenshots/README.md`（内容如上）
4. `git add` **仅**这两个新文件（精确路径，避免卷入 `FireUp_App`→`FIREprint_App` 的 rename diff）
5. Commit：`chore: add en-US screenshot scaffold for FIREprint fastlane`

## 不做的事情（明确排除）

- **不处理** `FireUp_App` → `FIREprint_App` 的目录改名 git diff（那是一个独立的大变更，需要用 `git add -A` + `git mv` 或手动梳理，不在本次范围）
- 不修改 Fastfile（保持 `skip_screenshots: true`，README 已注明未来如何启用）
- 不创建 `zh-Hans` / `zh-Hant` 或任何其他 locale 的 screenshots 目录
- 不做 `fastlane snapshot`（UI 自动化截图）配置，本次纯手动工作流
- 不编译 Swift 代码（无源码改动）