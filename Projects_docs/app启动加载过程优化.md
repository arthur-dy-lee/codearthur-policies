

## iOS 首次下载冷启动慢的原因

首次从 App Store 下载启动（Cold Launch）比后续启动慢，主要原因：

1. dyld 动态链接：首次需要加载所有动态库并做符号绑定，系统会缓存结果供后续使用
2. Runtime 初始化：ObjC/Swift metadata 注册、`+load` 方法、静态初始化器
3. 无磁盘缓存：首次无任何本地数据、图片缓存、模型文件
4. 隐私合规流程：UMP consent、ATT 弹窗等需要网络请求
5. 广告 SDK 初始化：TopOn/AdMob 等 SDK 首次初始化较慢

------

## 行业普遍做法

### 1. 系统 LaunchScreen 与自定义 Splash 无缝衔接（最核心）

这是最主流的做法——用一个自定义 Splash View 接管系统 LaunchScreen，让用户感知不到"卡住"：

系统 LaunchScreen（静态） → 自定义 SplashView（可动画） → 主页面

关键点：

- 自定义 SplashView 的首帧必须与系统 LaunchScreen 视觉一致（同背景色、同 Logo 位置），避免闪烁
- SplashView 上可以加 loading 动画（呼吸光效、进度环、品牌动画等），让等待有"进度感"
- 在 SplashView 展示期间，后台并行完成所有初始化任务

你的 SilenceCut 项目已经在用这个模式（`SplashView` + `showSplash` + `bootstrapIfNeeded`），FireUp 和 InnerRelease 还没有。

### 2. 启动任务分级 + 延迟加载

把启动任务分为三级：

| 级别    | 时机              | 内容                                     |
| :------ | :---------------- | :--------------------------------------- |
| P0 必须 | Splash 期间       | 隐私合规(UMP)、核心数据初始化            |
| P1 重要 | 主页出现后        | 广告 SDK 初始化、banner 预加载           |
| P2 延迟 | 用户稳定后 (3-6s) | interstitial/rewarded 预加载、非关键 SDK |

原则：只把阻塞用户看到主页面的最少任务放在 Splash 阶段，其余全部延迟。

### 3. 骨架屏 / Placeholder UI

主页面先用骨架屏（Skeleton）或 placeholder 内容展示，数据到了再替换：

- 列表页用灰色矩形占位
- 图片位置用渐变色块
- 让用户立刻看到"页面结构"，降低感知等待时间

### 4. 首次引导页（Onboarding）作为天然缓冲

很多 App 利用 Onboarding 引导流程（欢迎页、功能介绍、权限申请）作为启动缓冲：

- 用户在看引导页的 3-10 秒里，后台完成所有重初始化
- 引导完成时，主页面已完全就绪
- 你的三个项目都有 Onboarding，可以充分利用这个时间窗口

### 5. 预编译/预热优化

- App 瘦身：减少动态库数量（合并为静态库）、减少 ObjC `+load` 方法
- 预热关键路径：如 SilenceCut 已做的 `warmUpDeferredAdFormatsAfterUIStable()`
- 启动阶段避免同步网络请求、大文件 I/O

### 6. 启动耗时埋点

用 `os_signpost` 或自定义 trace 标记各阶段耗时，持续监控优化：

- SilenceCut 已有 `StartupTrace.mark` 系列埋点，这是好的实践

------

## 对你项目的具体建议

启动流程是否"系统 LaunchScreen\n(静态)""SplashView\n(品牌动画)""首次?""Onboarding\n(后台初始化)""主页面\n(骨架屏)"

- SilenceCut：已有较完善的 Splash + bootstrap 机制，建议确保 Onboarding 期间也在并行预加载广告
- FireUp / InnerRelease：目前用空 `UILaunchScreen {}`，没有自定义 Splash，建议补上 SplashView 过渡层，在其中完成 SwiftData 容器、隐私合规、广告初始化等

------

需要我针对某个具体项目制定详细的实施方案吗？比如为 FireUp 或 InnerRelease 添加 Splash 过渡机制？