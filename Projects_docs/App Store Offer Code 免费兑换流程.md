# App Store Offer Code 免费兑换流程

> 适用：matrixApps 旗下所有 app（SilenceCut / FinanceFreedom / HealthRhythm 等）通过 **Offer Code（优惠码）** 给指定账号免费或打折发放 IAP。
> 最后更新：2026-06。来源见文末。

---

## 0. 为什么用 Offer Code（而不是 Promo Code）

- **Promo Code（促销码）针对 IAP 的能力已被苹果停掉**：自 **2026-03-26** 起，App Store Connect **不能再为 IAP 创建 Promo Code**（已生成的可用到过期；给 *App 免费下载* 的 Promo Code 不受影响）。
- **替代品 = Offer Code**，已扩展到**全部 IAP 类型**：消耗型 / **非消耗型** / 非续期订阅 / 自动续期订阅。
- **关键**：对**非消耗型（Non-Consumable，如 lifetime 买断）**用 **Free Offer（免费）**，兑换后是**永久拥有**（一次性购买，不过期、不续订）。
  - “limited-time offer” 指的是**码的兑换期限**（见下），**不是拥有时长**。兑换后永久。

---

## 1. 前提条件

| 条件 | 要求 |
|---|---|
| App 状态 | **Ready for Distribution** |
| IAP 状态 | 该 IAP 必须 **Approved** |
| 例外 | **Sandbox 码**不受上面两条限制，可在审核前先测 |
| 角色权限 | ASC 中需有 Admin / App Manager（含 Offer Code 权限）、且已签 Paid Apps 协议 |

---

## 2. 如何创建 Free Offer + 生成码（ASC 步骤）

1. App Store Connect → **Apps** → 选 app
2. 侧边栏 → **In-App Purchases**
3. 点开目标 IAP（例：`com.codearthur.matrixApps.SilenceCut.pro.lifetime`）
4. 滚到 **Offer Codes** 区 → **Create Offer**
5. 填 **Reference Name**（内部名）
6. 选 **Customer Eligibility（兑换人群）**：
   - 尚未购买过 / 30 天内购买过 / 超过 30 天前购买过
   - 👉 给“没买过的指定账号送免费”：选**尚未购买过**那档
7. 选 **Countries or Regions（可用国家/地区）** —— ⚠️ 见第 5 节地区约束
8. 选 **Free Offer**（而非 Paid Offer）→ 确认
9. 生成码（三种）：
   - **One-time-use（一次性码）**：每码一次
   - **Custom（自定义码）**：一个码多次兑换
   - **Sandbox（沙盒码）**：测试用

---

## 3. 码的类型 / 额度 / 过期

| 项 | One-time-use | Custom | Sandbox |
|---|---|---|---|
| 每季度上限（每 app，全 IAP 共享） | 最多 100 万 | 最多 100 万 | 最多 1 万 |
| 单批 | 500 – 25,000 / 批（可多批） | 单码最多 25,000 次兑换（可加批） | — |
| 兑换 URL | ✓ | ✓ | — |
| 过期 | **必填，最长 6 个月** | 可选，设了则 ≤6 个月 | 可选 |
| 每人 | 每个 offer 限兑 **1 个码** | 同左 | — |

> 过期时间点：到期日 **12:00 a.m. PT**。
> ⚠️ 再强调：过期只限制“多久内要兑换”；**非消耗型兑换后拥有是永久的**。

---

## 4. 如何兑换（三种渠道）

1. **兑换 URL（推荐，免代码）**
   - ASC 生成码时提供；格式大致 `https://apps.apple.com/redeem?ctx=offercodes&id=<APP_APPLE_ID>&code=<CODE>`
   - 可用邮件 / 消息 / 浏览器打开，**不需要改 app 代码**，也不必在 App Store 里手动找“兑换”菜单
2. **App Store 直接输码**
   - App Store → 头像 → “兑换充值卡或代码”
3. **应用内兑换**
   - 需 **iOS 16.3+ / macOS 15.0+ / visionOS 1.0+**，且需写代码调起兑换 sheet
     - StoreKit 2：`AppStore.presentOfferCodeRedeemSheet(in:)`
     - 旧：`SKPaymentQueue.default().presentCodeRedemptionSheet()`
   - 👉 不想改代码就**不用这条**，用兑换 URL 即可

**归属**：兑换码归**兑换时登录 App Store 的那个 Apple ID**。要给账号 A、B 各自免费，就让 A、B **各自登录自己的 App Store** 去兑各自的码；之后 app 通过 `Transaction.currentEntitlements` 认出该账号拥有该 IAP。

---

## 5. ⚠️ 地区约束（硬限制，换渠道也绕不过）

- Offer Code **绑 storefront（账号所在国家/地区）**。兑换账号必须满足：
  1. 其 **App Store 账号地区**是该 **app + IAP 可用**的地区；
  2. 且你创建 Free Offer 时**勾选了该地区**。
- **跨区兑不了**：礼品卡/兑换码不能跨国家地区兑换；offer/promo code 只在“用户 profile 所设的同一 App Store”生效，**即便 app 全球分发**。
- 换兑换渠道（URL / App Store / 应用内）**都不能绕过**——最终都进“当前登录 Apple ID 的 storefront”。

### 如果账号不在 app 的上架区，仍要永久免费？
Offer Code 做不到。需改用**身份本地授予（SIWA 白名单）**：
- 把目标账号的 **Sign in with Apple 的 `user` 标识符**（形如 `000810.<hash>.0121`）打包进 app 的白名单；
- 设备做一次「通过 Apple 登录」→ app 拿到 id → 命中名单 → **本地直接授予 Pro**，不经 StoreKit、不看 storefront；
- SIWA 绑**开发者团队**而非地区 → 任何地区的该 Apple ID 都能解锁；别人登录拿到的是自己的 id，不在名单内 → 不会被白嫖。
- 现实前提：app 仍需先装到该设备（上架区 / **TestFlight 跨区邀请**）。
- 参考 FinanceFreedom 的 `LifetimeAwareEntitlementProvider` + `lifetime_allowlist.json` 实现。

---

## 6. 标准操作流程（以发放永久免费买断为例）

```
1. ASC 建非消耗型 lifetime IAP
2. 代码里把该 productID 接进 lifetimeProductIDs（确保 app 能认出拥有态）
3. 发一个包含该 IAP 的 app 版本，连同 IAP 一起提审
4. 上架 + IAP Approved
5. ASC → 该 IAP → Offer Codes → Create Offer → Free Offer → 选地区/人群 → 生成码
6. 把兑换 URL 发给目标账号 → 各自登录自己的 App Store 兑换
7. app 通过 currentEntitlements 认出 → 永久 Pro
```

> 想要**跨上架区**的账号也免费 → 走第 5 节的 SIWA 白名单，与 Offer Code 可并存。

---

## 参考

- Apple Developer News（2025-10-29 公告）：https://developer.apple.com/news/?id=gf6mgrs6
- App Store Connect Help — Create offer codes for In-App Purchases：https://developer.apple.com/help/app-store-connect/manage-in-app-purchases/create-offer-codes-for-in-app-purchases/
- Apple Support — 礼品卡/兑换码不能跨国家地区兑换：https://support.apple.com/en-us/108285
