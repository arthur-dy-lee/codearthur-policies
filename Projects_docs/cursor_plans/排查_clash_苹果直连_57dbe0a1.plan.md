---
name: 排查 Clash 苹果直连
overview: 定位 Clash Verge Rev 下苹果站点在 macOS 上“开着 Clash 直连打不开、走代理或关闭 Clash 又正常”的原因，并给出一套更稳的规则与排查顺序。重点区分规则命中问题和 TUN/DNS/fake-ip 导致的解析问题。
todos:
  - id: fix-rule-conflict
    content: 先修正 `apple.com` 被写成 `PROXY` 的规则冲突，改为苹果基础域名与 CDN 直连
    status: pending
  - id: separate-tun-dns
    content: 区分系统代理模式与 TUN 模式，优先判断是否为 macOS 下 TUN/DNS/fake-ip 导致的解析失败
    status: pending
  - id: simplify-dns
    content: 简化 Clash DNS 配置，并视情况把苹果关键域名加入 `fake-ip-filter` 进行验证
    status: pending
  - id: prepare-merge-samples
    content: 根据用户是否开启 TUN，准备一份更稳的 Merge 配置示例
    status: pending
isProject: false
---

# 排查 Clash 苹果直连

## 结论先行

你现在的现象通常不是单纯“苹果直连不行”，而是两类问题叠加：

1. 规则本身有冲突：你当前配置里写的是 `DOMAIN-SUFFIX,apple.com,PROXY`，所以 `apple.com` 根本不是直连，而是被你强制走代理。
2. 即使把苹果域名改成 `DIRECT`，在 `Clash Verge Rev + macOS + TUN` 场景下，也可能因为 `DNS`、`fake-ip`、`strict-route` 等问题导致“规则命中了 DIRECT，但域名解析失败”，表现为不开 Clash 正常、开 Clash 就打不开。

## 重点排查顺序

### 1. 先修正规则冲突

先把最明显的冲突排掉：

```yaml
- DOMAIN-SUFFIX,apple.com,PROXY
```

这条如果保留，`apple.com` 永远不会直连。应改成：

```yaml
- DOMAIN-SUFFIX,apple.com,DIRECT
```

同时保留这些通常没问题：

```yaml
- DOMAIN-SUFFIX,apple.com.cn,DIRECT
- DOMAIN-SUFFIX,icloud.com,DIRECT
- DOMAIN-SUFFIX,mzstatic.com,DIRECT
- DOMAIN-SUFFIX,aaplimg.com,DIRECT
- DOMAIN-SUFFIX,cdn-apple.com,DIRECT
- IP-CIDR,17.0.0.0/8,DIRECT
```

### 2. 不要把“Apple 全家桶”一刀切理解成同一类流量

苹果服务里有两类：

- 官网、文档、静态资源、App Store CDN：通常直连更稳
- 某些国际化服务、个别接口、国外 Apple 边缘节点：在部分网络环境下直连会慢、超时、或握手异常

所以“苹果必须全部直连”并不总是成立。更准确的目标应是：

- `App Store / iCloud / CDN / 静态资源` 尽量直连
- 如果某个子域名在你所在网络直连确实不稳，再单独放代理，而不是把整个 `apple.com` 都代理或都直连

### 3. 重点怀疑 DNS，而不是规则

根据 Clash Verge Rev 的公开 issue，macOS 上很常见的是：

- 规则已经命中 `DIRECT`
- 但日志仍报 `dns resolve failed: couldn't find ip`
- 关闭 Clash 后恢复正常

这说明流量不是“被错误代理”，而是 Clash 接管 DNS 后解析链路异常。

### 4. 如果你开了 TUN，优先怀疑 `fake-ip` / `strict-route`

公开反馈里，`Clash Verge Rev` 在部分 macOS 版本与 TUN 组合下，存在：

- TUN 开启后 DNS 被强制或偏向 `fake-ip`
- `strict-route: false` 时 DNS 查询可能回环
- 结果是直连域名也解析失败

这会造成一种错觉：

- 走代理反而能开，因为代理链路里 DNS 解析绕过去了
- 关掉 Clash 也能开，因为系统 DNS 恢复正常
- 唯独“开 Clash + 直连”最容易暴露问题

## 建议的调整方案

### 方案 A：先用最稳的思路验证

如果你的目标是“开 Clash 时苹果直连能正常用”，最稳的验证顺序是：

1. 把 `apple.com` 从 `PROXY` 改成 `DIRECT`
2. 暂时关闭 TUN，只保留系统代理模式
3. 用最简 DNS 配置测试，不要先上复杂的国外 DoH / fallback / fake-ip 组合
4. 确认 `apple.com`、`icloud.com`、`apps.apple.com`、`mzstatic.com` 能打开后，再决定是否重新开启 TUN

这样可以先区分：

- 是规则问题
- 还是 TUN/DNS 问题

### 方案 B：如果必须开 TUN

如果你必须用 TUN，那么重点不是继续堆规则，而是处理 DNS：

- 尽量避免过度复杂的 DNS 配置
- 如果当前订阅里自带很多国外 DoH / fallback，先简化
- 优先测试只保留稳定可达的 DNS
- 如支持配置 `fake-ip-filter`，可把苹果关键域名加入排除，避免 fake-ip 干扰

建议优先加的思路：

```yaml
fake-ip-filter:
  - '+.apple.com'
  - '+.apple.com.cn'
  - '+.icloud.com'
  - '+.mzstatic.com'
  - '+.aaplimg.com'
  - '+.cdn-apple.com'
```

注意：这不是万能药，但在“规则已直连、实际仍打不开”的场景里很值得先试。

### 方案 C：若仍异常，优先简化 DNS

很多类似案例最后不是靠加更多规则解决，而是靠把 Clash 自带 DNS 简化，甚至临时禁用 Clash DNS 验证问题来源。

排查时可按这个顺序：

1. 先保留苹果直连规则
2. 简化或临时禁用 Clash DNS
3. 看苹果站点是否恢复
4. 如果恢复，说明根因在 DNS 接管链路，不在分流规则本身

## 更适合你的配置策略

你的目标不是“苹果全部直连”，而是“开着 Clash 时，苹果常用服务正常直连”。更建议采用：

- 苹果基础域名与 CDN：`DIRECT`
- AI、Google、Telegram：`PROXY`
- 国内站点：`GEOSITE,CN,DIRECT` 和 `GEOIP,CN,DIRECT`
- 避免同时叠加过多 Apple 规则、复杂 DNS、TUN fake-ip，再去判断问题，否则很难知道到底是哪一层坏了

## 判断标准

如果调整后出现以下情况，基本可判断方向：

- `系统代理模式` 正常、`TUN 模式` 异常：优先查 TUN / DNS
- `关闭 Clash` 正常、`开启 Clash + DIRECT` 异常：优先查 Clash DNS 接管
- `开启 Clash + PROXY` 正常：不代表苹果应该走代理，只说明代理链路绕过了你当前的 DNS/TUN 问题

## 我下一步建议

下一轮可以直接按你的 Clash Verge 实际配置方式，帮你输出一份更稳的 `Merge` 示例，分两版：

- `系统代理优先版`：最稳，适合先验证
- `TUN 保守版`：尽量兼顾苹果直连与 AI 代理

如果你愿意，我再根据你是否开启了 `TUN`、当前 `dns:` 配置完整内容，给你定制成一份可直接粘贴的配置。