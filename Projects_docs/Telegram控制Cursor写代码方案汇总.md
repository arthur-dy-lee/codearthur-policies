# Telegram 控制 Cursor 写代码 —— 方案汇总

> 创建日期：2026-03-29
> 状态：调研完成

---

## 方案总览

| 方案 | 核心原理 | 需要本地在线 | 额外订阅 | 成熟度 | 当前可用 |
|------|---------|:----------:|:-------:|:-----:|:-------:|
| ① Cursor Cloud + cursor-tg | Cursor Cloud Agent API | ❌ | 无（Cursor 计划内） | ⭐⭐⭐⭐ | ✅ |
| ② Claude Code Channels | Claude Code CLI + MCP | ✅ | Claude Pro/Max | ⭐⭐⭐⭐ | ❌ |
| ③ TeleCode | pyautogui 操控本地 Cursor GUI | ✅ | 无 | ⭐⭐⭐ | ✅ |
| ④ Telegram MCP Server | MCP 协议桥接 | ✅ | 无 | ⭐⭐ | ⚠️ 组件级 |

---

## 方案一：Cursor Cloud Telegram Connector（cursor-tg）

### 概述

通过 Python 服务桥接 Cursor Cloud Agents API 与 Telegram Bot，实现在 Telegram 中创建、管理、监控 Cursor 云端 Agent，审批并合并 PR。

### 项目地址

- GitHub: https://github.com/tb5z035i/cursor-tg

### 架构

```
Telegram Bot ──→ cursor-tg (Python) ──→ Cursor Cloud Agents API
                     │                          │
                     │                          ├─ clone GitHub 仓库
                     │                          ├─ 云端 VM 中编码 + 测试
                     │                          └─ 创建 PR
                     │
                     ├─ SQLite（Agent 状态持久化）
                     └─ GitHub API（PR 操作）
```

### 代码提交流程

1. Telegram 发送指令（通过 `/newagent` 向导）
2. cursor-tg 调用 Cursor Cloud API 创建 Agent
3. Agent 在云端 VM 中 fresh clone 你的 GitHub 仓库
4. 自动创建特性分支（如 `cursor/add-readme-1234`）
5. 在云端完成编码、运行测试
6. 自动创建 PR（附截图、日志、验证材料）
7. Telegram 收到通知，可直接查看 diff、审批、合并

### 前置条件

- Python 3.12+
- Telegram Bot Token（BotFather 创建）
- Cursor API Key（Cursor Dashboard 获取）
- 项目托管在 GitHub，Cursor 已授权 read-write
- Cursor 计划需开启 on-demand usage
- 可选：GitHub Fine-grained PAT（用于 PR diff/merge 操作）

### Bot 命令一览

| 命令 | 功能 |
|------|------|
| `/newagent` | 创建新 Agent（4 步向导：模型、仓库、分支、prompt） |
| `/agents` | 列出所有运行中/已完成的 Agent |
| `/current` | 查看当前活跃 Agent 信息 |
| `/history` | 回放最近 N 条对话消息 |
| `/focus` | 切换活跃 Agent |
| `/stop` | 停止当前运行的 Agent |
| `/pr` | 查看 PR 状态和操作按钮 |
| `/diff` | 在 Telegram 中查看 PR diff |
| `/ready` | 标记 PR 为 ready for review |
| `/merge` | 合并 PR（支持 merge/squash/rebase） |
| `/threadmode` | 开启/关闭每 Agent 独立话题 |
| `/configure_unread` | 设置未读消息行为（全文/计数/静音） |

### 优点

- 不需要本地电脑在线，完全云端运行
- 天然支持 GitHub PR 工作流
- 可多 Agent 并行，每个任务隔离
- 有完整的状态持久化和消息追踪

### 缺点

- Cloud Agent 按 Max Mode API 价格计费，成本较高
- 项目必须托管在 GitHub/GitLab
- 无法操作本地未提交的代码

### 费用

Cursor Cloud Agent 使用 Max Mode 计费，按 API 调用量收费，具体价格参考 Cursor 官方定价页。

---

## 方案二：Claude Code Channels

### 概述

Anthropic 官方在 Claude Code v2.1.80+ 推出的 Channels 功能。通过 MCP 协议，将 Telegram 消息推送到运行中的 Claude Code 会话，实现双向通信。

### 架构

```
Telegram Bot ──→ Claude Code Channels (MCP Server)
                     │
                     ├─ 接收 Telegram 消息作为 channel event
                     ├─ Claude Code 执行编码任务
                     └─ 结果通过 Telegram 回复
```

### 设置步骤

```bash
# 1. 确认版本
claude --version  # 需要 v2.1.80+

# 2. 安装 Bun 运行时
curl -fsSL https://bun.sh/install | bash

# 3. 在 Claude Code 中安装 Telegram 插件
/plugin install telegram@claude-plugins-official

# 4. 配置 Bot Token
/telegram:configure <your-bot-token>

# 5. 启动（带 channels 标志）
claude --channels plugin:telegram@claude-plugins-official

# 6. Telegram 中发消息给 Bot，获取配对码
/telegram:access pair <code>

# 7. 设置仅允许自己发送
/telegram:access policy allowlist
```

### 工作流程

1. Telegram 发消息给 Bot
2. 消息到达本地 Claude Code 终端（作为 channel event）
3. Claude Code 处理任务（读/写文件、执行命令）
4. 结果通过 Telegram 回复

### 前置条件

- **Claude Pro/Max 订阅**（claude.ai，与 Cursor 订阅完全独立）
- Claude Code CLI v2.1.80+
- Bun 运行时
- Telegram Bot Token
- 本地 Claude Code 必须保持运行

### 安全机制

- Sender Allowlist：仅配对的 Telegram ID 可发送消息
- Per-session opt-in：必须显式 `--channels` 启动
- 企业管理员可全局启停

### 优点

- 官方支持，最原生的方案
- 轻量，双向通信
- 直接操作本地文件系统

### 缺点

- 需要单独的 Claude Pro/Max 订阅（$20-200/月）
- 本地 Claude Code 必须保持运行
- 权限提示需要本地审批（除非 `--dangerously-skip-permissions`）
- 控制的是 Claude Code，不是 Cursor IDE

### 当前不可用原因

需要单独的 Anthropic Claude 订阅和 Claude Code CLI，与 Cursor 订阅无关。

---

## 方案三：TeleCode（语音驱动）

### 概述

开源桌面应用，通过 pyautogui 屏幕自动化技术直接操控本地 Cursor IDE。支持语音转代码，零额外 API 费用。

### 项目地址

- 官网: https://telecodebot.com/
- GitHub: https://github.com/flexfinRTP/telecode

### 架构

```
Telegram（语音/文字）──→ TeleCode 桌面应用
                            │
                            ├─ 语音转写（如有）
                            ├─ pyautogui 操控 Cursor GUI
                            ├─ 实时截图推送回 Telegram
                            └─ Diff 审批 → 接受/拒绝
```

### 工作流程

1. Telegram 发送语音消息或文字指令
2. TeleCode 转写语音（如需），生成编码 prompt
3. 通过 pyautogui 将 prompt 输入 Cursor
4. 实时截图发回 Telegram，展示 AI 编码过程
5. 完成后发送 diff 摘要
6. 在 Telegram 中一键审批或拒绝更改

### 安装

```bash
# 方式一：从 Release 下载安装包（推荐，无需 Python）
# https://github.com/flexfinRTP/telecode/releases

# 方式二：源码安装
git clone https://github.com/flexfinrtp/telecode.git
cd telecode && ./setup.sh
```

### 前置条件

- 本地安装 Cursor IDE
- 本地电脑保持开机（显示器可关）
- Telegram Bot Token
- 无需额外 API 费用（复用 Cursor 订阅）

### 支持的模型

Claude Opus/Sonnet/Haiku、GPT-4、Gemini 等（通过 Cursor 支持的模型）

### 操作命令

- Keep All / Undo All / Continue / Run / Stop
- 与 Cursor 原生操作一致

### 安全机制

- Token 加密存储（DPAPI/Keychain）
- 单用户认证（仅你的 Telegram ID）
- Prompt 注入防护
- 文件系统沙箱
- 无开放端口（Telegram 长轮询，仅出站）
- 完整审计日志

### 优点

- 零额外 API 费用，复用 Cursor 订阅
- 支持语音输入
- 实时截图反馈，可视化编码过程
- 真正操控 Cursor GUI，体验一致
- 开源，MIT 协议

### 缺点

- 本地电脑必须保持开机
- 依赖屏幕自动化（pyautogui），稳定性受系统环境影响
- macOS 版标注为 "coming soon"（需确认最新状态）
- 非结构化操控，可能偶尔出错

---

## 方案四：Telegram MCP Server

### 概述

通过 MCP（Model Context Protocol）协议在 Cursor 中配置 Telegram Server，让 Cursor Agent 能收发 Telegram 消息。这是一个组件级方案，需要自行组合成完整工作流。

### 参考

- MCPCursor: https://mcpcursor.com/server/telegram-1-mcp

### 架构

```
Cursor IDE ←→ MCP Server (Telegram) ←→ Telegram Bot API
```

### 配置方式

在 `.cursor/mcp.json` 中添加 Telegram MCP Server 配置：

```json
{
  "mcpServers": {
    "telegram": {
      "command": "npx",
      "args": ["-y", "telegram-mcp-server"],
      "env": {
        "TELEGRAM_BOT_TOKEN": "your-bot-token",
        "TELEGRAM_ALLOWED_USERS": "your-telegram-id"
      }
    }
  }
}
```

### 前置条件

- Cursor IDE（支持 MCP）
- Telegram Bot Token
- Node.js 运行时
- 本地 Cursor 保持运行

### 优点

- 轻量，配置简单
- 原生 MCP 协议，与 Cursor AI 深度集成
- 灵活，可自定义工作流

### 缺点

- 仅是通信组件，不含完整工作流（无 PR 管理、无截图反馈）
- 需要自行开发上层逻辑
- 本地 Cursor 必须保持运行
- 社区维护，成熟度较低

---

## 与 matrixAppsCore 集成的可能性

现有 matrixAppsCore 架构已经具备良好基础：

| 已有组件 | 路径 | 可复用点 |
|---------|------|---------|
| Telegram Bot | `bot/telegram_bot.py` | 消息接收和路由 |
| Claude Code Bridge | `plugins/claude_code_bridge.py` | 代码执行接口模式 |
| Code Executor 接口 | `plugins/code_executor.py` | 新增 Cursor Cloud 实现 |
| Workflow Engine | `orchestrator/engine.py` | 编排多 Agent 流程 |
| Mode Router | `dispatcher/mode_router.py` | 路由到 Task/Workflow 模式 |

### 集成路线（推荐）

在 `plugins/` 下新增 `cursor_cloud_bridge.py`，实现 `CodeExecutor` 接口，调用 Cursor Cloud Agents API：

```
Telegram 消息
  → matrixAppsCore dispatcher
  → Coder Agent（cursor_cloud_bridge）
  → Cursor Cloud API
  → GitHub PR
  → Telegram 通知审批
```

这样可以复用 matrixAppsCore 的完整 Agent 编排能力（PM 分析需求 → Coder 写代码 → QA 验证），后端从 Claude Code CLI 切换或增加 Cursor Cloud API。

---

## 选型建议

| 场景 | 推荐方案 |
|------|---------|
| 远程控制，不需要本地在线 | ✅ 方案一：cursor-tg |
| 零成本，本地有电脑在线 | ✅ 方案三：TeleCode |
| 深度集成到 matrixAppsCore | ✅ 扩展方案：cursor_cloud_bridge |
| 已有 Claude 订阅，追求轻量 | 方案二：Claude Code Channels |
| 需要灵活定制 MCP 工作流 | 方案四：Telegram MCP |
