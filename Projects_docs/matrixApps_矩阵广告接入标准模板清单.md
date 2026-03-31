# matrixApps 矩阵广告接入标准模板清单

## 结论先说

可以预埋，但不要一次性全家桶。建议采用“高概率 + 低风险”的分层预埋策略。

“后台开关无缝启用”成立的前提是：
- 对应 network adapter 已打包进客户端
- `Info.plist` 配置完整
- 隐私声明和合规材料已覆盖
- `SKAdNetworkItems` 已补齐

对 matrixApps 的最佳实践：统一广告基座 + 每个 App 只配参数，不在每个 App 重复接 SDK。

## 三步策略（逐条建议）

### 1) 预埋主流 SDK（可行）

首批建议预埋 3-4 家：
- Google
- Pangle
- Meta
- AppLovin

接入方式建议：
- 统一通过 TopOn adapter 管理，不把各家 SDK 调用散落在业务代码中
- 任何新增 network，都记录两类性能基线：
  - IPA 体积增量
  - 启动耗时增量

### 2) 后台控制开关（可行）

上线初期建议只开 Google 流量，其他 network 全部关闭，先验证主漏斗。

注意：
- “关闭流量”不等于“无合规义务”
- 只要 SDK 在包内，仍可能触发隐私声明与审核披露要求

### 3) 后续无缝开启竞价（基本可行）

前提是客户端已包含：
- 目标网络 adapter
- 该网络所需配置项（AppID/Placement/隐私配置等）

如果上述任一项缺失，仍需要发版。iOS 不允许动态下载可执行代码来补 SDK。

## 必须提前完成的 6 件事

1. **隐私合规一次性设计**
   - App Privacy
   - ATT
   - GDPR/UMP
   - 中国区合规

2. **`Info.plist` 与 `SKAdNetworkItems` 清单完备**
   - 预埋网络的 IDs 在首版即补齐

3. **Remote Config 开关体系**
   - 分层维度：国家 / App 版本 / 用户分群
   - 开关层级：全局 / Network / 广告位

4. **统一日志与监控**
   - 最低指标：fill rate、eCPM、展示成功率、崩溃率
   - 要求可追溯到具体 network 维度

5. **降级与熔断**
   - 某 network 异常时，后台可一键熔断
   - 支持自动降级到保底 network

6. **发布前检查单**
   - 每次新增 network，至少完成四项检查：
     - 包体
     - 启动性能
     - 隐私合规
     - 审核风险

## 针对 matrixApps 的最省人力落地方案

### 架构原则

将 `TopOnAdapter` 抽到 `Shared` 的广告模块（建议命名 `MyAppAdsMediation`）。

每个 App 仅保留：
- AppID / AppKey
- 各广告位 Placement ID
- 区域策略（例如：CN 用 TopOn，Global 用 AdMob/TopOn）

### 模板工程默认内建

- `Podfile`：按可选 adapter 列表分层注释
- 自动 `pod install` 脚本：新项目一键初始化
- `.xcworkspace` 打开约束：避免误开 `.xcodeproj`

## 标准发布检查清单（可直接执行）

每次接入或新增 network，按以下顺序执行：

1. 依赖检查
   - adapter 是否已编译进包
   - 版本是否与基座兼容

2. 配置检查
   - `Info.plist` 配置是否齐全
   - `SKAdNetworkItems` 是否完整
   - 各 network key / placement 是否区分环境

3. 合规检查
   - ATT 触发路径是否正确
   - UMP/GDPR 同意流程是否可回放
   - App Store Connect 隐私问卷是否同步更新

4. 质量检查
   - 冷启动时间是否超阈值
   - 广告加载成功率是否达标
   - crash 是否可归因到 network / adapter

5. 灰度检查
   - 首发只开单 network（建议 Google）
   - 远程开关可按国家与版本精细控制
   - 异常时可 5 分钟内完成熔断

## 变更原则

- 不在业务模块直接写第三方 SDK 调用
- 不在未补齐合规材料前开启 network
- 不在无监控与熔断能力时做多 network 全量放开
