---
name: Add sample video import
overview: 将样例视频打包进 App Bundle，DEBUG 启动时自动拷贝到 Documents 目录，让模拟器中通过现有"浏览文件"流程即可导入。
todos:
  - id: add-resource-yml
    content: 在 project.yml 中添加样例视频资源引用 + UIFileSharingEnabled
    status: completed
  - id: add-copy-on-launch
    content: 在 SilenceCutApp.swift 的 bootstrapIfNeeded 中添加 DEBUG 拷贝逻辑
    status: in_progress
  - id: regenerate-project
    content: 运行 xcodegen generate 重新生成 Xcode 项目
    status: pending
isProject: false
---

# 让模拟器中可通过现有流程导入样例视频

## 背景

模拟器中没有视频文件可选，用户无法通过现有的"浏览文件"功能导入视频，导致无法截图。需要将样例视频放到模拟器的 Files app 可访问的位置，使现有导入流程正常工作。

## 方案

核心思路：将样例视频打包进 App Bundle，在 DEBUG 启动时自动拷贝到 App 的 Documents 目录，并开启 `UIFileSharingEnabled` 使 Documents 目录在 Files app 中可见。用户通过现有的"浏览文件"按钮即可在 "On My iPhone > SilenceCut" 中找到该视频。**不修改任何 UI 或导入逻辑。**

### 1. 在 project.yml 中添加样例视频资源 + UIFileSharingEnabled

修改 [project.yml](Projects/SilenceCut_App/ios_workspace/project.yml)：

- 在 `SilenceCutApp` target 添加 `resources` 引用样例视频文件：

```yaml
    resources:
      - path: ../docs/silenceCut_explame_video.mp4
        buildPhase: resources
```

- 在 `info.properties` 中添加两个键，让 Documents 目录在 Files app 中可见：

```yaml
        UIFileSharingEnabled: true
        LSSupportsOpeningDocumentsInPlace: true
```

### 2. 在 SilenceCutApp.swift 启动时自动拷贝样例视频到 Documents

修改 [SilenceCutApp.swift](Projects/SilenceCut_App/ios_workspace/SilenceCutApp/SilenceCutApp.swift)，在 `bootstrapIfNeeded()` 顶部添加 `#if DEBUG` 逻辑：

```swift
#if DEBUG
private func copySampleVideoToDocumentsIfNeeded() {
    guard let bundleURL = Bundle.main.url(forResource: "silenceCut_explame_video", withExtension: "mp4") else { return }
    let docs = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
    let dest = docs.appendingPathComponent("silenceCut_explame_video.mp4")
    guard !FileManager.default.fileExists(atPath: dest.path) else { return }
    try? FileManager.default.copyItem(at: bundleURL, to: dest)
}
#endif
```

在 `bootstrapIfNeeded()` 开头调用此方法。

### 3. 重新生成 Xcode 项目

运行 `xcodegen generate` 重新生成项目文件。

## 用户操作流程

1. 在模拟器中运行 App
2. 启动时样例视频自动拷贝到 Documents
3. 点击首页右上角 `+` > "浏览文件"
4. 在 Files app 中导航到 "On My iPhone" > "SilenceCut"
5. 选择 `silenceCut_explame_video.mp4`
6. 正常走完现有导入流程

## 删除方式

后续删除时：

1. 从 `project.yml` 移除 `resources` 条目 + `UIFileSharingEnabled` / `LSSupportsOpeningDocumentsInPlace`
2. 从 `SilenceCutApp.swift` 移除 `#if DEBUG` 的 `copySampleVideoToDocumentsIfNeeded` 方法及调用
3. 重新 `xcodegen generate`

