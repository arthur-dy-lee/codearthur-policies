# codearthur-policies

Arthur Lee 在 Apple App Store 上架的所有 App 的法律合规文档。通过 GitHub Pages 托管。

## 在线地址

```
https://arthur-dy-lee.github.io/codearthur-policies/
```

## 目录结构

```
codearthur-policies/
├── main/
│   ├── dongyue/           # 对应 Dongyue Li 的账号
│   │   ├── en/
│   │   │   ├── privacy.html
│   │   │   ├── terms.html
│   │   │   └── account-deletion.html
│   │   └── zh/            # 如果有中文版
│   │
│   └── bozhu/             # 对应 bozhu.li 的账号
│       ├── en/
│       │   ├── privacy.html
│       │   ├── terms.html
│       │   └── account-deletion.html
│       └── zh/
├── LICENSE
└── README.md
```

## 文档说明

| 文件 | EN | ZH | 说明 |
|:--|:--|:--|:--|
| `privacy.html` | 20 条 | 20 条 | 隐私政策 |
| `terms.html` | 30 条 | 30 条 | 用户协议 & EULA（合并） |
| `account-deletion.html` | 8 条 | 8 条 | 账号与数据删除说明 |

中英文章节编号一一对应，内容一致。英文版额外包含 GDPR/CCPA 合规细节引用。

---

### privacy.html — 隐私政策 (20 条)

| 章节 | 涵盖内容 |
|:--|:--|
| 1. 信息收集 | 用户主动提供的信息（账号、用户内容、客服沟通）、自动收集的信息（设备信息、使用数据、崩溃日志）、第三方服务收集的信息（广告 IDFA、分析数据） |
| 2. 不收集数据的 App | 纯工具类 App 不收集/传输/存储任何个人数据的声明 |
| 3. 必要数据收集与功能限制 | 拒绝授权可能导致功能不可用、开发者不对此担责、GDPR 撤回同意权说明 |
| 4. 信息使用方式 | 功能维护、Bug 修复、广告展示、客服沟通、欺诈/滥用检测、法律合规与执法响应；不出售/出租/交易用户数据 |
| 5. 信息披露 | 法律要求/权利保护/执法请求/业务转让/服务提供商共享的披露场景；服务商须签署数据处理协议 (GDPR Art. 28) |
| 6. 广告与水印 | 免费版含广告和水印、第三方广告网络（AdMob）数据采集、付费去除为自愿行为 |
| 7. Apple ATT | iOS 14.5+ 追踪授权说明、用户可随时更改追踪偏好 |
| 8. 数据存储与安全 | 本地存储优先、iCloud 同步走 Apple 安全体系、合理安全措施但无法保证绝对安全 |
| 9. 数据保留 | 按需保留、本地数据随 App 删除而清除、账号数据可随时请求删除 |
| 10. 第三方服务 | 第三方服务有各自隐私政策、开发者不对第三方隐私实践负责、第三方服务变更可能导致功能调整 |
| 11. AI 功能 | 输入数据可能发送至第三方 AI 服务商处理、仅供参考不构成专业建议、不使用用户数据训练模型 |
| 12. 用户权利 | 访问/更正/删除/导出/撤回同意/拒绝个性化广告 |
| 13. 账号与数据删除 | App 内删除操作说明、删除范围（本地 + 云端）、不可恢复警告 |
| 14. 儿童隐私 | 不面向 13 岁以下儿童、不主动收集儿童数据 |
| 15. 跨境数据传输 | 数据可能跨境传输、用户使用即视为同意 |
| 16. EEA 用户 (GDPR) | 处理依据：同意/正当利益/合同履行；可向当地数据保护机构投诉 |
| 17. 加州用户 (CCPA) | 知情权/删除权/拒绝出售权/不歧视权；声明不出售个人信息 |
| 18. 政策变更 | 通过更新日期通知、重大变更可能 App 内提示、继续使用视为接受 |
| 19. 免责声明 | 不可控因素导致的数据泄露免责、第三方隐私实践免责、AS IS 声明 |
| 20. 联系方式 | 开发者名称与邮箱 |

### terms.html — 用户协议 & EULA (30 条)

| 章节 | 涵盖内容 |
|:--|:--|
| 1. 许可授权 | 有限、非独占、不可转让、可撤销的许可；许可而非出售；开发者保留所有知识产权 |
| 2. 使用限制 | 禁止复制/修改/分发/逆向工程/去除版权声明/非法使用/自动化抓取 |
| 3. 免费版、广告与水印 | 免费版含广告和水印、付费去除为自愿行为、核心功能免费可用 |
| 4. 订阅与内购 | Apple 处理支付、自动续订规则、取消方式、退款由 Apple 处理 |
| 5. 用户内容与知识产权 | 用户拥有其内容、用户对内容合法性负全责、反向赔偿条款、开发者可删除违规内容并封禁用户 |
| 6. 系统升级与前向兼容性免责 | 不保证与未来 iOS/设备兼容、音视频框架等 API 被废弃的具体例子、开发者有绝对权利决定是否适配 |
| 7. 第三方服务与 API 免责 | LLM/云存储/支付等中断免责、API Key 被吊销免责、成本过高可关停功能、开源依赖 breaking change |
| 8. AI 功能免责 | AI 输出仅供参考、不构成任何专业建议、用户自行验证、AI 不可用不担责 |
| 9. 服务可用性与中断 | 不保证 7x24 可用、服务器宕机/维护/流量过载免责、个人开发者无 SLA 义务 |
| 10. 安全与数据泄露 | 黑客/DDoS/零日漏洞/第三方被攻破免责、数据泄露不担责、用户自行维护设备安全 |
| 11. 数据丢失 | App 更新/系统更新/iCloud 同步失败/数据库损坏导致数据丢失均免责、用户自行备份 |
| 12. 不可抗力 | 自然灾害/战争/疫情/政府制裁/网络攻击/Apple 政策变更；持续 90 天以上可终止协议 |
| 13. App 停止开发与下架 | 随时可停止更新/下架/关服务器/转让所有权、不退款不赔偿 |
| 14. 跨境使用风险 | 受限地区法律后果自负、跨境数据传输风险、当地税费自理 |
| 15. 保证免责 | AS IS / AS AVAILABLE、不保证无错/不中断/安全/结果准确/缺陷会修复 |
| 16. 赔偿上限 | 不承担间接/附带/特殊/后果性/惩罚性损害；最大赔偿额为 12 个月付费金额或 $50 取低 |
| 17. 账号封禁与服务拒绝 | 任何时候任何理由可拒绝服务/限制/冻结/永久注销账号、无需退款或赔偿 |
| 18. 功能降级与有限服务 | 可关闭/限制功能、免费功能可转付费、可调整订阅价格、可按地区限制服务 |
| 19. 实验性与 Beta 功能 | Beta 功能不保证稳定、可随时移除、可能导致数据丢失、用户自愿承担风险 |
| 20. 设备与系统要求 | 旧设备/旧系统不保证可用、性能下降或崩溃不担责 |
| 21. 开源组件 | 开源库按现状提供、不对开源组件的漏洞/缺陷/不兼容担责 |
| 22. 集体诉讼放弃 | 用户只能以个人身份索赔、放弃集体诉讼权利（不影响欧盟消费者法定权利） |
| 23. 不弃权 | 未执行权利不代表放弃权利 |
| 24. 协议转让 | 开发者可不经同意转让协议（含合并/收购/资产出售场景） |
| 25. Apple 强制条款 | 协议仅在用户与开发者之间、Apple 不是当事方、Apple 无维护/支持/保修义务、Apple 为第三方受益人 |
| 26. 管辖法律与争议解决 | 以开发者所在地法律为准、先协商 30 天再诉讼 |
| 27. 协议修改 | 随时可修改、更新日期标注、继续使用视为接受 |
| 28. 可分割性 | 单条无效不影响其余条款效力 |
| 29. 完整协议 | 本协议 + 隐私政策构成完整协议 |
| 30. 联系方式 | 开发者名称与邮箱 |

### account-deletion.html — 账号与数据删除说明 (8 条)

| 章节 | 涵盖内容 |
|:--|:--|
| 1. 删除方式 | 方式 A：App 内设置 > 删除账号；方式 B：发邮件请求删除（30 天内处理、特殊情况可延长、需身份验证） |
| 2. 删除范围 | 账号凭证、用户内容、本地数据、iCloud 云端数据、偏好设置、App 内操作记录 |
| 3. 不删除的内容 | App Store 购买记录（Apple 管理）、已匿名化的统计数据、法律要求保留的数据 |
| 4. 订阅提醒 | 删除账号不等于取消订阅、需手动在设备设置中取消以避免继续扣费 |
| 5. 不可恢复警告 | 删除操作永久且不可逆、建议提前导出/备份重要数据 |
| 6. 删除后重新注册 | 数据不可恢复到新账号、权益不可转移、可拒绝违规用户重新注册 |
| 7. 无账号 App | 卸载 App 即清除本地数据、iCloud 数据可在设备设置中手动删除 |
| 8. 联系方式 | 开发者名称与邮箱 |

---

## URL 规则

所有文档支持通过 URL 参数动态注入 App 名称，一套文档服务矩阵中的所有 App：

```
# 英文版
https://arthur-dy-lee.github.io/codearthur-policies/bozhu/en/privacy.html?name=FireApp
https://arthur-dy-lee.github.io/codearthur-policies/bozhu/en/terms.html?name=FireApp
https://arthur-dy-lee.github.io/codearthur-policies/bozhu/en/account-deletion.html?name=FireApp

# 中文版
https://arthur-dy-lee.github.io/codearthur-policies/bozhu/zh/privacy.html?name=FireApp
https://arthur-dy-lee.github.io/codearthur-policies/bozhu/zh/terms.html?name=FireApp
https://arthur-dy-lee.github.io/codearthur-policies/bozhu/zh/account-deletion.html?name=FireApp
```

支持两种参数名：`?name=AppName` 或 `?app=AppName`，效果一致。页面内的 JS 会自动替换文档中的 App 名称占位符。

## App 中的调用方式

### Swift 示例代码

```swift
import SafariServices

struct PolicyURLs {
    static let baseURL = "https://arthur-dy-lee.github.io/codearthur-policies"

    /// 根据系统语言选择 en 或 zh
    static var lang: String {
        let preferred = Locale.preferredLanguages.first ?? "en"
        return preferred.hasPrefix("zh") ? "zh" : "en"
    }

    static func privacyURL(appName: String) -> URL {
        URL(string: "\(baseURL)/\(lang)/privacy.html?name=\(appName)")!
    }

    static func termsURL(appName: String) -> URL {
        URL(string: "\(baseURL)/\(lang)/terms.html?name=\(appName)")!
    }

    static func accountDeletionURL(appName: String) -> URL {
        URL(string: "\(baseURL)/\(lang)/account-deletion.html?name=\(appName)")!
    }
}

// 使用示例：在设置页中打开
func openPrivacyPolicy() {
    let url = PolicyURLs.privacyURL(appName: "FireApp")
    let safari = SFSafariViewController(url: url)
    present(safari, animated: true)
}
```

### App Store Connect 提交审核时

在 App Store Connect > App 信息 中填入：

| 字段 | 填入值 |
|:--|:--|
| Privacy Policy URL | `https://arthur-dy-lee.github.io/codearthur-policies/bozhu/en/privacy.html?name=你的AppName` |

### App 内设置页建议展示

```
设置
├── 隐私政策 / Privacy Policy      → 打开 privacy.html
├── 用户协议 / Terms of Service    → 打开 terms.html
└── 删除账号 / Delete Account      → 打开 account-deletion.html（如有账号功能）
```

## 法律合规覆盖

| 地区 | 法规 | 覆盖位置 |
|:--|:--|:--|
| 全球 | Apple App Store 审核要求 | 全文 |
| 欧盟 27 国 | GDPR | privacy.html 第 16 条, terms.html 第 22 条 |
| 美国加州 | CCPA | privacy.html 第 17 条 |
| 美国（联邦） | COPPA（儿童隐私） | privacy.html 第 14 条 |
| 所有地区 | Apple ATT | privacy.html 第 7 条 |
| 所有地区 | Apple 账号删除要求 (2023+) | account-deletion.html |

## 启用 GitHub Pages

1. 打开 https://github.com/arthur-dy-lee/codearthur-policies
2. 点 **Settings**（顶部 tab 栏最右边）
3. 左侧菜单找到 **Pages**（在 "Code and automation" 分类下）
4. 在 **"Build and deployment"** 区域，**Source** 下拉框选 Deploy from a branch
5. **Branch** 选 main，文件夹选 / (root)
6. 点 **Save**

## 更新流程

修改仓库中的 HTML 文件 → 推送至 GitHub → GitHub Pages 自动部署 → App 内立即生效（无需发版）。

重大条款变更时建议通过 App 内弹窗通知用户。