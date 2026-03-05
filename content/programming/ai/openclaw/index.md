---
title: "OpenClaw"
type: "docs"
weight: 3
---

OpenClaw 是一个**自托管的 AI 网关**，让你可以通过 WhatsApp、Telegram、Discord、iMessage 等聊天软件随时与 AI 助手对话。你在自己的机器上运行一个 Gateway 进程，它成为消息应用与 AI 之间的桥梁。

官方文档：[docs.openclaw.ai](https://docs.openclaw.ai/)

---

## 一、核心概念

```
手机 / 电脑上的聊天 App
         ↕
  ┌─── Gateway ───┐   ← 你机器上的后台进程
  │  Channel 接入  │     WhatsApp / Telegram / Discord …
  │  Agent 调度   │   ← 一个或多个 AI "大脑"
  │  Session 存储  │   ← 每段对话的历史记录
  └───────────────┘
         ↕
    Tools / Skills     ← AI 的能力层
```

| 概念 | 是什么 |
|------|--------|
| **Gateway** | 整个系统的唯一后台进程，管理渠道连接、Agent 调度、Web 控制台 |
| **Channel** | 接入的聊天平台（WhatsApp、Telegram、Discord、iMessage…） |
| **Agent** | 一个完全隔离的 AI"大脑"，有独立的人格、工作目录、会话历史 |
| **Session** | 一段对话的历史记录，可按发送者/群组/渠道隔离 |
| **Tool** | AI 可调用的原子能力（即 Function Calling），如读文件、执行命令、搜网页 |
| **Skill** | 一个 `SKILL.md` 文件，教 AI 某类工具怎么用，在对话开始时注入 Prompt |
| **Sub-Agent** | 主 Agent 在后台派生的子任务实例，完成后将结果公告回主会话 |
| **Plugin** | npm 包形式的扩展，可增加新渠道或新工具 |

---

## 二、快速上手（5 分钟）

### 1. 安装

```bash
# macOS / Linux
curl -fsSL https://openclaw.ai/install.sh | bash

# Windows (PowerShell)
iwr -useb https://openclaw.ai/install.ps1 | iex

# 或 npm
npm install -g openclaw@latest
```

### 2. 引导向导

```bash
openclaw onboard --install-daemon
```

向导会交互式地完成：API Key 配置、Gateway 设置、可选渠道接入。

### 3. 检查并打开控制台

```bash
openclaw gateway status   # 确认 Gateway 在运行
openclaw dashboard        # 浏览器打开 http://127.0.0.1:18789/
```

控制台无需任何渠道配置，直接就能在网页上与 AI 对话。

---

## 三、接入聊天渠道

配置文件统一位于 `~/.openclaw/openclaw.json`（JSON5 格式，支持注释）。

### WhatsApp

```bash
openclaw channels login --channel whatsapp   # 扫码授权
```

```json5
// openclaw.json
{
  channels: {
    whatsapp: {
      allowFrom: ["+8613800138000"]  // 白名单，留空则全部拒绝
    }
  }
}
```

### Telegram

1. 用 [@BotFather](https://t.me/BotFather) 创建 Bot，拿到 Token
2. 写入配置：

```json5
{
  channels: {
    telegram: {
      botToken: "123456:ABC...",
      dmPolicy: "pairing"   // pairing | allowlist | open | disabled
    }
  }
}
```

### Discord

1. 在 [Discord 开发者平台](https://discord.com/developers) 创建 Bot，开启 **Message Content Intent**
2. 写入配置：

```json5
{
  channels: {
    discord: {
      accounts: {
        default: {
          token: "DISCORD_BOT_TOKEN",
          guilds: {
            "服务器ID": {
              channels: { "频道ID": { allow: true, requireMention: false } }
            }
          }
        }
      }
    }
  }
}
```

### 访问控制策略

所有渠道的私信都受 `dmPolicy` 控制：

| 值 | 效果 |
|----|------|
| `pairing`（默认） | 未知用户收到配对码，需手动审批 |
| `allowlist` | 只有 `allowFrom` 列表中的用户可发消息 |
| `open` | 所有人均可（需同时设 `allowFrom: ["*"]`） |
| `disabled` | 屏蔽所有私信 |

群组消息默认需要 @mention 才响应，可在 Agent 配置中自定义触发词。

> 配置文件支持**热重载**——保存后 Gateway 自动应用，大多数设置无需重启。

---

## 四、Agent 管理

### 工作目录文件

每个 Agent 有一个工作目录（默认 `~/.openclaw/workspace`），里面的 Markdown 文件在每次对话开始时注入 Prompt：

| 文件 | 注入内容 |
|------|----------|
| `AGENTS.md` | 操作指令与"记忆"（最重要） |
| `SOUL.md` | 人格、语气、边界 |
| `USER.md` | 用户档案与称呼偏好 |
| `TOOLS.md` | 工具使用提示（不控制工具是否存在） |
| `IDENTITY.md` | Agent 名字、风格 |

```bash
openclaw setup   # 初始化工作目录，生成默认模板文件
```

### 添加与查看

```bash
openclaw agents add work          # 创建隔离的新 Agent
openclaw agents list              # 列出所有 Agent
openclaw agents list --bindings   # 同时显示路由规则
```

### 多 Agent 配置

一台机器可以运行多个完全隔离的 Agent，每个 Agent 有独立的工作目录、认证档案、会话存储。

**场景 A：个人号 / 工作号各一个 Agent**

```json5
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work", model: "anthropic/claude-opus-4-6" }
    ]
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } }
  ]
}
```

**场景 B：WhatsApp 用快速模型，Telegram 用深度思考模型**

```json5
{
  agents: {
    list: [
      { id: "chat", workspace: "~/.openclaw/workspace-chat", model: "anthropic/claude-sonnet-4-5" },
      { id: "opus", workspace: "~/.openclaw/workspace-opus", model: "anthropic/claude-opus-4-6" }
    ]
  },
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } }
  ]
}
```

消息路由采用**最精确优先**原则：精确 peer > 父线程 > 频道账号 > 渠道通配 > 默认 Agent。

---

## 五、Tools 与 Skills

### Tools（工具 / Function Calling）

Tool 是 AI 可调用的原子能力，通过 JSON Schema 传给模型 API（即通常说的 Function Calling）。工具通过两个并行通道呈现给 AI：一是系统 Prompt 中的说明文字，二是传给模型的结构化 Schema——两者缺一不可。

**内置工具分组：**

| 分组 | 包含工具 | 说明 |
|------|----------|------|
| `group:fs` | `read` `write` `edit` `apply_patch` | 文件读写编辑 |
| `group:runtime` | `exec` `bash` `process` | 执行命令、管理后台进程 |
| `group:web` | `web_search` `web_fetch` | 搜索网页、抓取页面内容 |
| `group:ui` | `browser` `canvas` | 控制浏览器、驱动移动端 Canvas |
| `group:messaging` | `message` | 跨渠道发送消息 |
| `group:sessions` | `sessions_*` | 管理会话、查看历史 |
| `group:memory` | `memory_search` `memory_get` | 检索记忆 |
| `group:nodes` | `nodes` | 控制配对的移动设备 |
| `group:automation` | `cron` `gateway` | 定时任务、Gateway 控制 |
| —— | `image` `pdf` `sessions_spawn` | 图像分析、PDF 解析、派生子 Agent |

**工具权限配置：**

```json5
// 全局工具策略
{
  tools: {
    profile: "coding",       // minimal | coding | messaging | full
    allow: ["browser"],      // 额外开启某个工具
    deny: ["group:runtime"]  // 禁止某类工具
  }
}
```

```json5
// 为特定 Agent 单独设置（deny 优先于 allow）
{
  agents: {
    list: [{
      id: "family",
      tools: {
        allow: ["read", "sessions_list"],
        deny: ["exec", "write", "edit", "apply_patch"]
      }
    }]
  }
}
```

### Skills（技能）

Skill 本质上是**放在特定目录下的 `SKILL.md` 文件**，在每次对话开始时被注入系统 Prompt，告诉 AI"你有这个能力，这样用"。它不新增工具本身，而是提供使用说明。

**加载优先级（高 → 低）：**

```
<workspace>/skills/     ← 当前 Agent 专属
~/.openclaw/skills/     ← 所有 Agent 共享
[内置 Skills]           ← 随 OpenClaw 安装
```

同名 Skill 高优先级覆盖低优先级。

**创建一个自定义 Skill：**

```bash
# 1. 创建目录
mkdir -p ~/.openclaw/workspace/skills/my-skill

# 2. 编写说明文件
cat > ~/.openclaw/workspace/skills/my-skill/SKILL.md << 'EOF'
---
name: my-skill
description: 调用内部 API 查询数据
metadata: {"openclaw": {"requires": {"bins": ["curl"], "env": ["MY_API_KEY"]}}}
---

当用户查询订单时，使用 exec 工具运行：
curl -H "Authorization: $MY_API_KEY" https://api.example.com/orders
EOF

# 3. 重启使其生效
openclaw gateway restart
```

**`SKILL.md` frontmatter 说明：**

| 字段 | 必填 | 说明 |
|------|------|------|
| `name` | ✓ | 唯一标识符 |
| `description` | ✓ | 简短描述，AI 用来决定是否调用 |
| `user-invocable` | — | 是否作为 `/skill-name` 斜杠命令暴露（默认 true） |
| `metadata.openclaw.requires.bins` | — | 必须存在于 PATH 的二进制，否则跳过加载 |
| `metadata.openclaw.requires.env` | — | 必须存在的环境变量，否则跳过加载 |
| `metadata.openclaw.requires.os` | — | 限定操作系统（`darwin` / `linux` / `win32`） |

不满足条件的 Skill 自动跳过，不注入 Prompt，不消耗 token。

**为 Skill 配置 API Key：**

```json5
// openclaw.json
{
  skills: {
    entries: {
      "my-skill": {
        enabled: true,
        apiKey: "sk-...",            // 对应 primaryEnv 字段
        env: { MY_API_KEY: "xxx" }  // 注入但不覆盖已有的环境变量
      }
    }
  }
}
```

**从 [ClawHub](https://clawhub.com) 安装社区 Skill：**

```bash
clawhub install <skill-name>   # 安装到 workspace/skills/
clawhub update --all           # 更新所有已安装的 Skills
```

---

## 六、会话管理

### 查看会话

```bash
openclaw sessions                    # 默认 Agent 的会话
openclaw sessions --agent work       # 指定 Agent
openclaw sessions --all-agents       # 所有 Agent
openclaw sessions --active 120       # 最近 120 分钟内活跃的会话
```

### 会话隔离范围

通过 `dmScope` 控制私信的会话如何隔离：

| 值 | 效果 |
|----|------|
| `main`（默认） | 所有私信共用一个会话 |
| `per-peer` | 每个发送者独立会话 |
| `per-channel-peer` | 每个渠道 + 发送者独立会话（多用户场景推荐） |
| `per-account-channel-peer` | 最细粒度隔离 |

```json5
{ session: { dmScope: "per-channel-peer" } }
```

### 自动重置与清理

```json5
// 每天凌晨 4 点自动重置，或空闲 2 小时后重置
{
  session: {
    reset: { mode: "daily", atHour: 4, idleMinutes: 120 }
  }
}
```

```bash
openclaw sessions cleanup --dry-run          # 预览待清理内容
openclaw sessions cleanup --all-agents --enforce   # 实际执行清理
```

---

## 七、进阶功能

### Sub-Agent（后台子任务）

Sub-Agent 是主 Agent 派生的独立后台运行实例，用于**并行处理耗时任务**，完成后将结果公告回主会话。

```
主 Agent  →  sessions_spawn  →  子 Agent（独立会话、独立 token）
                                      ↓ 完成后
                              公告结果到主会话频道
```

在对话中直接说"帮我派生一个子任务"或使用斜杠命令：

```
/subagents list              # 查看子 Agent 列表
/subagents spawn <任务描述>   # 手动派生
/subagents kill <runId>      # 停止
/subagents log <runId>       # 查看日志
```

嵌套子 Agent（编排模式，默认关闭）：

```json5
{
  agents: { defaults: { subagents: { maxSpawnDepth: 2, maxConcurrent: 8 } } }
}
```

### 沙箱隔离

为不受信任的 Agent 启用 Docker 沙箱，限制其工具权限：

```json5
{
  agents: {
    list: [{
      id: "untrusted",
      workspace: "~/.openclaw/workspace-untrusted",
      sandbox: { mode: "all", scope: "agent" },
      tools: { allow: ["read"], deny: ["exec", "write", "edit"] }
    }]
  }
}
```

### 定时任务（Cron）

```json5
{ cron: { enabled: true, maxConcurrentRuns: 2 } }
```

启用后可通过 AI 对话或 `openclaw cron` 命令管理定时触发的 Agent 任务。

### Webhook 自动化

让外部服务（如 Gmail 推送）触发 Agent 运行：

```json5
{
  hooks: {
    enabled: true,
    token: "your-secret",
    mappings: [{ match: { path: "gmail" }, action: "agent", agentId: "main" }]
  }
}
```

### 移动端节点（Nodes）

配对 iOS / Android 设备后，Agent 可调用：摄像头、麦克风、设备位置、屏幕录制、系统通知等。

---

## 八、CLI 速查

```bash
# Gateway
openclaw gateway status / restart
openclaw gateway --port 18789    # 前台调试模式
openclaw doctor [--fix]          # 诊断并修复配置问题
openclaw logs                    # 查看日志

# 渠道
openclaw channels login [--channel whatsapp] [--account biz]
openclaw channels status [--probe]

# Agent
openclaw agents list [--bindings]
openclaw agents add <id>
openclaw setup                   # 初始化工作目录

# 会话
openclaw sessions [--agent <id>] [--all-agents] [--active 120]
openclaw sessions cleanup [--dry-run] [--enforce]

# 配置
openclaw onboard / configure
openclaw config get <key>
openclaw config set <key> <value>

# 模型
openclaw models list

# 发消息
openclaw message send --target +8613800138000 --message "Hello"
```

---

## 九、MCP（Model Context Protocol）

MCP 是由 Anthropic 主导、已捐赠给 Linux 基金会的开放标准，提供一套统一协议让 AI 客户端连接任意外部工具、数据库与 API。OpenClaw 原生支持 MCP，充当 **MCP Client**，而各类 MCP Server 则作为能力桥接层。

### MCP 与内置工具的对比

| 维度 | 内置工具 / Skills | MCP Server |
|------|-----------------|------------|
| 接入方式 | 随 OpenClaw 安装或 SKILL.md | 独立进程，标准协议 |
| 使用场景 | 个人工作流、OpenClaw 专属逻辑 | 对接外部 API / 数据库 / 云服务 |
| 复用性 | 限 OpenClaw 生态 | 任意 MCP 兼容客户端可复用 |
| 典型用法 | Skills 内部调用 MCP Server | GitHub、Notion、PostgreSQL… |

### 安装 MCP Server

**方式一：CLI 一键安装（内置支持的服务）**

```bash
openclaw mcp install github
openclaw mcp install notion
openclaw mcp install postgres
```

**方式二：在 `openclaw.json` 中手动配置**

```json5
// ~/.openclaw/openclaw.json
{
  mcpServers: {
    github: {
      command: "npx",
      args: ["-y", "@modelcontextprotocol/server-github"],
      env: {
        GITHUB_TOKEN: "ghp_xxxxxxxxxxxx"
      }
    },
    notion: {
      command: "node",
      args: ["/path/to/notion-mcp-server/index.js"],
      env: {
        NOTION_API_KEY: "secret_xxxxxxxxxxxx"
      }
    },
    postgres: {
      command: "npx",
      args: ["-y", "@modelcontextprotocol/server-postgres"],
      env: {
        POSTGRES_CONNECTION_STRING: "postgresql://user:pass@localhost:5432/mydb"
      }
    }
  }
}
```

**方式三：从社区获取（ClawHub / GitHub）**

```bash
# ClawHub 上的 MCP Skill
clawhub install <mcp-skill-name>

# 官方 MCP Server 仓库
# github.com/modelcontextprotocol/servers
```

### 连接类型

| 类型 | 说明 | 典型场景 |
|------|------|---------|
| `stdio` | 本地子进程，标准输入输出 | 大多数本地 Server（默认） |
| `http` | REST API 端点 | 云端 / 远程 Server |
| `websocket` | 实时双向连接 | 需要推送通知的服务 |
| `ssh` | 远程服务器连接 | 内网 / 跳板机场景 |

### 常用 MCP Server 一览

| 类别 | Server | 能力 |
|------|--------|------|
| 开发 | `github` | 仓库管理、Issue、PR |
| 开发 | `gitlab` | CI/CD、合并请求 |
| 开发 | `jira` | 项目追踪、工单管理 |
| 生产力 | `notion` | 页面读写、数据库查询 |
| 生产力 | `slack` | 频道消息、工作空间 |
| 生产力 | `google-workspace` | Gmail、Calendar、Drive |
| 数据库 | `postgres` | SQL 查询、Schema 管理 |
| 数据库 | `mysql` | 数据库操作 |
| 数据库 | `mongodb` | 文档查询、聚合 |
| 云服务 | `aws` | S3、Lambda、EC2 |
| 云服务 | `google-cloud` | BigQuery、Cloud Storage |
| 搜索 | `tavily` | AI 搜索 |
| 爬虫 | `firecrawl` | 网页抓取 |
| 浏览器 | `playwright` | 浏览器自动化 |

### 使用示例

配置好后无需额外操作，OpenClaw 会自动感知已挂载的 MCP Server 并在合适时调用：

```
你：帮我在 openclaw 仓库创建一个 Issue，标题是「支持自定义主题」
AI：[调用 GitHub MCP Server → 创建 Issue] ✓ 已创建 #42
```

```
你：查询数据库中本月新增的用户
AI：[调用 PostgreSQL MCP Server → 执行 SQL] 共 128 条记录…
```

### 查看与调试

```bash
openclaw mcp list              # 查看已配置的 MCP Server
openclaw logs                  # 查看 MCP 调用日志
openclaw doctor                # 检查 MCP 配置是否正确
```

### 自定义 MCP Server（Node.js 示例）

```javascript
#!/usr/bin/env node
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';

const server = new Server(
  { name: 'my-server', version: '1.0.0' },
  { capabilities: { tools: {} } }
);

// 声明工具
server.setRequestHandler('tools/list', async () => ({
  tools: [{
    name: 'query_orders',
    description: '查询订单信息',
    inputSchema: {
      type: 'object',
      properties: {
        userId: { type: 'string', description: '用户 ID' }
      },
      required: ['userId']
    }
  }]
}));

// 处理工具调用
server.setRequestHandler('tools/call', async ({ params }) => {
  const { name, arguments: args } = params;
  if (name === 'query_orders') {
    const data = await fetchOrders(args.userId);  // 你的业务逻辑
    return { content: [{ type: 'text', text: JSON.stringify(data) }] };
  }
  throw new Error(`Unknown tool: ${name}`);
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

开发完成后，将其加入 `openclaw.json` 的 `mcpServers` 即可：

```json5
{
  mcpServers: {
    "my-server": {
      command: "node",
      args: ["/path/to/my-server/index.js"],
      env: { MY_API_KEY: "xxx" }
    }
  }
}
```

---

## 十、运维

### 配置 AI 模型

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["openai/gpt-5"]   // 主模型不可用时自动切换
      }
    }
  }
}
```

支持 Anthropic、OpenAI、OpenRouter、Ollama（本地）、Qwen、Moonshot 等。

### 远程访问

推荐通过 **Tailscale** 或 SSH 隧道访问：

```bash
ssh -N -L 18789:127.0.0.1:18789 user@your-server
```

### 故障排查

```bash
openclaw doctor           # 自动诊断，输出具体错误字段
openclaw doctor --fix     # 尝试自动修复
openclaw health           # 健康检查
openclaw logs             # 完整日志
openclaw channels status --probe   # 检查渠道连接
```

常见原因：配置文件格式错误（用 `doctor` 定位）、`allowFrom` 白名单未填写、渠道 Token 失效。

### 部署方式

| 方式 | 适合场景 |
|------|----------|
| 本地守护进程（默认） | 个人开发机，长期运行 |
| Docker | 服务器容器化部署 |
| Fly.io / Railway / Render | 无服务器云托管 |
| VPS（Hetzner / DigitalOcean / GCP） | 自管云服务器 |
| Raspberry Pi | 低功耗家用服务器 |
