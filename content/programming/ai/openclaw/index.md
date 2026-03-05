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

配置文件支持**热重载**——保存后 Gateway 自动应用，大多数设置无需重启。

---

## 四、Agent 管理

### 工作目录与初始化

每个 Agent 有一个**工作目录**（默认 `~/.openclaw/workspace`），这是 AI 的"家"，也是文件工具的默认 cwd。配置、会话历史、凭证等位于 `~/.openclaw/`，**不在**工作目录中。

```bash
openclaw setup          # 在当前工作目录生成缺失的模板文件（不覆盖已有文件）
openclaw setup --workspace ~/.openclaw/workspace-work   # 为指定路径生成模板
```

**初始化之后如何修改：** 直接用文本编辑器编辑工作目录下对应的 `.md` 文件即可，保存后下次对话自动生效，无需重启。如果某个文件被误删，再次运行 `openclaw setup` 只会补全缺失文件，不覆盖已有内容。若不想让 OpenClaw 自动生成引导文件，可在配置中禁用：

```json5
{ agent: { skipBootstrap: true } }
```

### 工作目录文件详解（系统 Prompt 注入体系）

工作目录里的 Markdown 文件并非普通笔记——它们构成 AI 每次对话的**系统 Prompt**。每次会话开始时，OpenClaw 按优先级将这些文件拼入上下文，AI 读取后才开始回复。

| 文件 | 何时加载 | 作用 | 类比 |
|------|---------|------|------|
| `AGENTS.md` | 每次会话开始 | **操作规程**：启动时读哪些文件、记忆管理流程、群聊行为、安全边界 | 岗位手册 |
| `SOUL.md` | 每次会话开始 | **系统 Prompt 核心**：人格、语气、价值观、不可逾越的边界 | 人格底层 |
| `USER.md` | 每次会话开始 | **用户档案**：称呼偏好、背景信息、习惯 | 用户画像 |
| `TOOLS.md` | 每次会话开始 | 工具使用提示与约定（仅指导，不控制工具是否存在） | 工具说明书 |
| `IDENTITY.md` | 每次会话开始 | Agent 的名字、主题风格、emoji | 名片 |
| `MEMORY.md` | 每次会话开始 | 长期记忆：决策、偏好、经验教训（由 AI 自动维护） | 长期记忆 |
| `memory/YYYY-MM-DD.md` | 会话开始读今天+昨天 | 每日日志：当天会话的原始记录 | 工作日志 |
| `BOOT.md` | Gateway 重启时 | 可选启动检查清单（内部钩子启用后触发）| 开机自启 |
| `HEARTBEAT.md` | 心跳运行时 | 极短的心跳任务提示（避免 token 浪费，保持简短） | 定时巡检 |
| `BOOTSTRAP.md` | 仅首次新工作目录 | 一次性初始化仪式，完成后删除 | 入职引导 |

> **文件大小限制：** 单文件超长会被截断。默认单文件上限 20,000 字符、所有文件合计 150,000 字符，可通过 `agents.defaults.bootstrapMaxChars` / `agents.defaults.bootstrapTotalMaxChars` 调整。

**最佳实践——SOUL.md 写法：**

```markdown
# Identity & Purpose
你是一个专注于后端开发的编程助手，熟悉 Go 和分布式系统。

# Security Boundaries
- 你绝对不能记录或输出任何 API Key、密码、私钥等凭证信息
- 未经明确授权，不得执行破坏性命令（rm -rf、DROP TABLE 等）

# Financial Boundaries
- 单次 API 调用预估费用超过 $1 时，必须先征得用户确认

# Operational Boundaries
- 不得修改 SOUL.md 本身，除非用户明确要求

# Escalation
遇到涉及生产数据库或外部支付接口的操作时，暂停并请求确认。
```

**最佳实践——AGENTS.md 写法（关键：会话启动流程）：**

```markdown
## Session Start（必须，优先于回复）
1. 读取 SOUL.md（我是谁）
2. 读取 USER.md（我在帮谁）
3. 读取 memory/YYYY-MM-DD.md（今天和昨天）
4. 读取 MEMORY.md（长期记忆）
5. 完成后直接开始，不要询问是否已完成

## Memory Rules
- 每次对话结束前，将重要决策、偏好、未完成的事项写入当天的 memory/ 文件
- 周期性将重要条目迁移到 MEMORY.md，删除过期的日志文件

## 安全规则
- 不要将私信内容转发到群组
- 群聊中只在真正有价值时发言
```

**推荐：将工作目录纳入 Git 管理（私有仓库）：**

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md SOUL.md USER.md TOOLS.md IDENTITY.md HEARTBEAT.md memory/
git commit -m "init workspace"
# 推送到私有仓库做备份
```

### 添加与管理 Agent（CLI）

**创建新 Agent：**

```bash
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw setup --workspace ~/.openclaw/workspace-work   # 为新 Agent 生成模板文件
```

**查看与删除：**

```bash
openclaw agents list              # 列出所有 Agent
openclaw agents list --bindings   # 同时显示路由绑定
openclaw agents delete work       # 删除 Agent（不删除工作目录文件）
```

**设置 Agent 身份（写入 IDENTITY.md 或直接覆盖配置）：**

```bash
# 从工作目录中的 IDENTITY.md 读取并写入配置
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity

# 直接用命令行覆盖字段
openclaw agents set-identity --agent main --name "Claw" --emoji "🦞" --avatar avatars/openclaw.png
```

### 多 Agent 渠道路由（CLI + 配置双轨）

**多 Agent 路由可以完全通过命令行配置，无需手动编辑 JSON 文件：**

```bash
# 为 work Agent 绑定 Telegram 的 ops 账号和 Discord 的 guild-a 服务器
openclaw agents bind --agent work --bind telegram:ops --bind discord:guild-a

# 查看当前所有绑定
openclaw agents bindings
openclaw agents bindings --agent work
openclaw agents bindings --json    # 输出 JSON，便于脚本处理

# 解绑
openclaw agents unbind --agent work --bind telegram:ops
openclaw agents unbind --agent work --all   # 解除该 Agent 的全部绑定
```

**绑定优先级规则（精确度高者优先）：**

```
精确 peer（特定用户）
  > 父线程
    > channel:accountId（特定账号）
      > channel:*（渠道通配）
        > 默认 Agent（default: true）
```

**配置文件方式（等效，适合版本控制）：**

```json5
// ~/.openclaw/openclaw.json
{
  agents: {
    list: [
      {
        id: "home",
        default: true,
        workspace: "~/.openclaw/workspace-home",
        identity: { name: "Home Claw", emoji: "🏠" }
      },
      {
        id: "work",
        workspace: "~/.openclaw/workspace-work",
        model: "anthropic/claude-opus-4-6",
        identity: { name: "Work Claw", emoji: "💼" }
      }
    ]
  }
}
```

**配置写入命令（非交互式）：**

```bash
# 通过 config set 修改单个字段（不推荐改复杂嵌套结构，建议直接编辑 JSON）
openclaw config set agents.defaults.workspace "~/.openclaw/workspace"
openclaw config get agents.list    # 查看当前 agents 列表
```

**典型场景一：个人号 / 工作号分离**

```bash
# 创建两个 Agent
openclaw agents add home --workspace ~/.openclaw/workspace-home
openclaw agents add work --workspace ~/.openclaw/workspace-work

# 绑定渠道
openclaw agents bind --agent home --bind whatsapp:personal
openclaw agents bind --agent work --bind whatsapp:biz --bind telegram:ops
```

**典型场景二：按渠道选用不同模型（需编辑 openclaw.json）**

```json5
{
  agents: {
    list: [
      { id: "chat", workspace: "~/.openclaw/workspace-chat", model: "anthropic/claude-sonnet-4-5" },
      { id: "deep", workspace: "~/.openclaw/workspace-deep", model: "anthropic/claude-opus-4-6" }
    ]
  }
}
```

```bash
openclaw agents bind --agent chat --bind whatsapp
openclaw agents bind --agent deep --bind telegram
```

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

## 六、MCP（Model Context Protocol）

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

### 用 Go 实现 MCP Server

Go 生态现有多个可用库：

| 库 | 说明 |
|----|------|
| [`modelcontextprotocol/go-sdk`](https://github.com/modelcontextprotocol/go-sdk) | 官方 Go SDK，与 Google 合作维护，支持最新规范 |
| [`mark3labs/mcp-go`](https://github.com/mark3labs/mcp-go) | 社区最流行实现，API 简洁，支持 stdio/SSE/HTTP 多种传输 |
| [`zeromicro/go-zero`](https://go-zero.dev) | go-zero 框架内置 MCP 模块，适合已有 go-zero 项目 |

#### 方案一：mcp-go（推荐独立使用）

```bash
go get github.com/mark3labs/mcp-go
```

```go
package main

import (
    "context"
    "fmt"

    "github.com/mark3labs/mcp-go/mcp"
    "github.com/mark3labs/mcp-go/server"
)

func main() {
    s := server.NewMCPServer("订单查询服务", "1.0.0",
        server.WithToolCapabilities(false),
    )

    // 定义工具及参数 Schema
    queryTool := mcp.NewTool("query_order",
        mcp.WithDescription("根据订单 ID 查询订单详情"),
        mcp.WithString("order_id",
            mcp.Required(),
            mcp.Description("订单 ID"),
        ),
    )
    s.AddTool(queryTool, queryOrderHandler)

    // 以 stdio 方式启动（供本地 MCP Client 调用）
    if err := server.ServeStdio(s); err != nil {
        fmt.Printf("server error: %v\n", err)
    }
}

func queryOrderHandler(ctx context.Context, req mcp.CallToolRequest) (*mcp.CallToolResult, error) {
    orderID, err := req.RequireString("order_id")
    if err != nil {
        return mcp.NewToolResultError(err.Error()), nil
    }
    // 替换为实际业务逻辑
    result := fmt.Sprintf(`{"order_id":"%s","status":"shipped","amount":299.0}`, orderID)
    return mcp.NewToolResultText(result), nil
}
```

**Struct 风格（更类型安全）：**

```go
type QueryOrderArgs struct {
    OrderID string `json:"order_id" jsonschema_description:"订单 ID" jsonschema:"required"`
    UserID  string `json:"user_id,omitempty" jsonschema_description:"用户 ID（可选过滤）"`
}

type OrderResult struct {
    OrderID string  `json:"order_id"`
    Status  string  `json:"status"`
    Amount  float64 `json:"amount"`
}

queryTool := mcp.NewTool("query_order",
    mcp.WithDescription("查询订单详情"),
    mcp.WithInputSchema[QueryOrderArgs](),
    mcp.WithOutputSchema[OrderResult](),
)
```

**HTTP/SSE 方式启动（供远程访问）：**

```go
// SSE 传输
if err := server.ServeSSE(s, ":8080"); err != nil {
    log.Fatal(err)
}

// Streamable HTTP 传输（新规范推荐）
httpServer := server.NewStreamableHTTPServer(s)
if err := httpServer.Start(":8080"); err != nil {
    log.Fatal(err)
}
```

接入 `openclaw.json`：

```json5
{
  mcpServers: {
    "order-service": {
      command: "go",
      args: ["run", "/path/to/mcp-server/main.go"],
      // 或编译后：command: "/path/to/order-mcp-server"
      env: { DB_DSN: "postgres://user:pass@localhost/orders" }
    }
  }
}
```

#### 方案二：go-zero 内置 MCP（推荐已有 go-zero 项目）

go-zero 从 v1.8+ 起内置 `mcp` 模块，基于 SSE 实现实时通信，支持工具、提示词（Prompt）、资源（Resource）三类能力。

```bash
go get github.com/zeromicro/go-zero@latest
```

**`config.yaml`：**

```yaml
Name: order-mcp-server
Host: localhost
Port: 8080
Mcp:
  Name: order-service
  MessageTimeout: 30s
  Cors:
    - http://localhost:18789   # OpenClaw Dashboard 地址
```

**`main.go`：**

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"

    "github.com/zeromicro/go-zero/core/conf"
    "github.com/zeromicro/go-zero/core/logx"
    "github.com/zeromicro/go-zero/mcp"
)

func main() {
    var c mcp.McpConf
    conf.MustLoad("config.yaml", &c)
    logx.DisableStat()

    s := mcp.NewMcpServer(c)
    defer s.Stop()

    registerTools(s)

    s.Start()
}

func registerTools(s *mcp.McpServer) {
    // 工具 1：查询订单
    s.AddTool(mcp.Tool{
        Name:        "query_order",
        Description: "根据订单 ID 查询订单详情",
        InputSchema: mcp.ToolInputSchema{
            Type: "object",
            Properties: map[string]mcp.Property{
                "order_id": {Type: "string", Description: "订单 ID"},
            },
            Required: []string{"order_id"},
        },
    }, func(ctx context.Context, req mcp.ToolRequest) (mcp.ToolResponse, error) {
        orderID := req.Params["order_id"].(string)
        data := map[string]any{
            "order_id": orderID,
            "status":   "shipped",
            "amount":   299.0,
        }
        b, _ := json.Marshal(data)
        return mcp.NewTextToolResponse(string(b)), nil
    })

    // 工具 2：列出用户订单
    s.AddTool(mcp.Tool{
        Name:        "list_orders",
        Description: "列出指定用户的订单列表",
        InputSchema: mcp.ToolInputSchema{
            Type: "object",
            Properties: map[string]mcp.Property{
                "user_id": {Type: "string", Description: "用户 ID"},
                "limit":   {Type: "integer", Description: "返回数量上限，默认 10"},
            },
            Required: []string{"user_id"},
        },
    }, func(ctx context.Context, req mcp.ToolRequest) (mcp.ToolResponse, error) {
        userID := req.Params["user_id"].(string)
        text := fmt.Sprintf("用户 %s 共有 3 条订单：#1001, #1002, #1003", userID)
        return mcp.NewTextToolResponse(text), nil
    })
}
```

接入 `openclaw.json`（HTTP/SSE 模式）：

```json5
{
  mcpServers: {
    "order-service": {
      "url": "http://localhost:8080/sse",
      "transport": "sse"
    }
  }
}
```

---

## 七、会话管理

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

## 八、进阶功能

### Sub-Agent（后台子任务）

Sub-Agent 是主 Agent 在后台派生的独立运行实例，拥有自己的会话和 token 用量，**非阻塞**——派生后立刻返回 `runId`，完成后将结果公告回主会话频道。

**会话层级：**

| 层级 | 会话 Key | 角色 | 能否再派生 |
|------|---------|------|-----------|
| 0 | `agent::main` | 主 Agent | 始终可以 |
| 1 | `agent::subagent:<runId>` | 子 Agent（默认） | 需开启 `maxSpawnDepth: 2` |
| 2 | `agent::subagent:...:subagent:<runId>` | 孙 Agent（叶节点） | 不可以 |

**方式一：通过对话直接触发（最简单）**

直接告诉 AI "帮我后台执行 XX 任务"，AI 会自动调用 `sessions_spawn`。

**方式二：斜杠命令**

```
/subagents spawn --model anthropic/claude-sonnet-4-5 --thinking extended
/subagents list                   # 查看当前所有子 Agent
/subagents info <runId>           # 查看运行状态、时间戳、会话 ID
/subagents log <runId> [limit]    # 查看输出日志
/subagents steer <runId>          # 向运行中的子 Agent 发送补充指令
/subagents send <runId>           # 向子 Agent 发送消息
/subagents kill <runId>           # 终止指定子 Agent（级联终止其子孙）
/subagents kill all               # 终止全部子 Agent
```

**方式三：AI 工具调用 `sessions_spawn`（编程式）**

```
sessions_spawn({
  task: "抓取 https://example.com 并总结主要内容，完成后公告结果",
  model: "anthropic/claude-sonnet-4-5",   // 可选，降低成本
  thinking: "extended",                    // 可选
  runTimeoutSeconds: 300,                  // 可选，超时自动终止
  label: "网页摘要任务",                   // 可选，便于识别
  cleanup: "delete"                        // 完成后自动归档
})
```

**开启嵌套子 Agent（编排模式）：**

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2,          // 允许子 Agent 再派生（默认 1）
        maxChildrenPerAgent: 5,    // 每个会话最多同时 5 个子任务
        maxConcurrent: 8,          // 全局并发上限
        runTimeoutSeconds: 900,    // 全局默认超时（0 = 不限）
        model: "anthropic/claude-sonnet-4-5"  // 子 Agent 默认使用更便宜的模型
      }
    }
  }
}
```

**编排模式示意（主 Agent → 一级编排 → 多个并行 Worker）：**

```
主 Agent
  └─ 编排子 Agent（maxSpawnDepth: 2）
       ├─ Worker 1（并行）
       ├─ Worker 2（并行）
       └─ Worker 3（并行）
```

每层只收到来自**直接子节点**的公告，结果逐层向上汇总。

> **注意：**
> - 子 Agent 只注入 `AGENTS.md` + `TOOLS.md`，**不注入** `SOUL.md`、`USER.md`、`IDENTITY.md`。
> - `/stop` 会终止主会话并**级联**终止所有子 Agent。
> - Gateway 重启后，未完成的子 Agent 公告任务会丢失。

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

## 九、运维

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

推荐通过 **Tailscale** 或 SSH 隧道访问。

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

---

## 十、CLI 速查

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
openclaw agents add <id> [--workspace <path>]
openclaw agents delete <id>
openclaw agents bind --agent <id> --bind <channel>[:<accountId>]
openclaw agents unbind --agent <id> --bind <channel>[:<accountId>] [--all]
openclaw agents bindings [--agent <id>] [--json]
openclaw agents set-identity --agent <id> [--name X] [--emoji X] [--avatar X]
openclaw agents set-identity --workspace <path> --from-identity
openclaw setup [--workspace <path>]  # 生成缺失的模板文件（不覆盖已有）

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
