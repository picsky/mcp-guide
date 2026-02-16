# Model Context Protocol (MCP) 开发指南

> 一份系统化、结构化的 MCP 开发文档，帮助你从入门到精通构建 MCP 服务器。

---

## 📚 目录

1. [概述篇](#一概述篇)
2. [基础篇](#二基础篇)
3. [实践篇](#三实践篇)
4. [进阶篇](#四进阶篇)
5. [案例篇](#五案例篇)
6. [资源汇总](#六资源汇总)

---

## 一、概述篇

### 1.1 什么是 MCP？

**Model Context Protocol (MCP)** 是由 **Anthropic** 于 **2024年11月** 开源的 AI 与外部世界连接的统一标准协议。

MCP 被形象地称为 **"AI 的 USB-C 接口"**，其核心目标是：

> **让大语言模型能够安全、标准化地调用外部工具和数据源。**

### 1.2 为什么需要 MCP？

在 MCP 出现之前，AI 应用与外部工具的集成面临以下问题：

| 问题 | 说明 |
|------|------|
| **碎片化** | 每个工具都有不同的 API 接口和认证方式 |
| **重复开发** | 每个 AI 应用都需要为相同工具重写集成代码 |
| **安全隐患** | 缺乏统一的安全标准和权限控制 |
| **维护困难** | 工具更新时，所有集成点都需要同步更新 |

**MCP 的解决方案**：
- ✅ 统一协议标准
- ✅ 一次开发，多处使用
- ✅ 内置安全机制
- ✅ 解耦工具提供者和 AI 应用

### 1.3 MCP 的核心概念

```
┌─────────────────────────────────────────────────────────┐
│                      MCP 架构图                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌──────────────┐      MCP 协议       ┌──────────────┐ │
│   │   AI 应用     │ ◄────────────────► │  MCP 服务器   │ │
│   │  (Claude等)   │   (stdio/SSE)      │  (工具提供)   │ │
│   └──────────────┘                      └──────────────┘ │
│          │                                     │        │
│          │ 发现工具                             │ 实现工具 │
│          │ 调用工具                             │ 数据库   │
│          │ 获取资源                             │ API     │
│          │                                     │ 文件系统 │
│   ┌──────▼──────┐                      ┌──────▼──────┐ │
│   │   Prompt    │                      │   Tools     │ │
│   │   Resources │                      │   Resources │ │
│   └─────────────┘                      └─────────────┘ │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

#### 核心组件

| 组件 | 说明 | 示例 |
|------|------|------|
| **Tools** | AI 可调用的函数 | 搜索、计算、数据库查询 |
| **Resources** | AI 可读取的数据 | 文件内容、数据库记录、API 响应 |
| **Prompts** | 预定义的交互模板 | 代码审查、数据分析流程 |

### 1.4 MCP 的应用场景

- 🤖 **AI 助手增强** - 让 Claude、GPT 等获得实时数据访问能力
- 🔧 **开发工具集成** - IDE、编辑器与 AI 的深度集成
- 📊 **数据分析** - 连接数据库、BI 工具进行智能分析
- 🌐 **自动化工作流** - 连接各种 SaaS 工具实现自动化
- 🔍 **知识管理** - 连接笔记、文档系统进行智能检索

---

## 二、基础篇

### 2.1 MCP 协议规范

#### 传输协议

MCP 支持两种传输方式：

| 方式 | 适用场景 | 特点 |
|------|----------|------|
| **stdio** | 本地进程通信 | 简单、安全、适合本地工具 |
| **SSE** | 远程服务器 | 支持网络通信、适合云服务 |

#### 消息格式

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "search",
    "arguments": {
      "query": "MCP tutorial"
    }
  }
}
```

### 2.2 开发环境搭建

#### 方式一：TypeScript（推荐）

```bash
# 安装官方创建工具
npm install -g @modelcontextprotocol/create-typescript-server

# 创建新项目
npx @modelcontextprotocol/create-typescript-server my-mcp-server

# 或使用 fastmcp 框架
npm install fastmcp
```

#### 方式二：Python

```bash
# 安装 Python SDK
pip install mcp

# 或使用官方 SDK
pip install modelcontextprotocol
```

#### 方式三：使用框架

| 框架 | 语言 | 特点 |
|------|------|------|
| [fastmcp](https://github.com/punkpeye/fastmcp) | TypeScript | 简洁、现代 |
| [mcp-framework](https://github.com/QuantGeekDev/mcp-framework) | TypeScript | 功能丰富 |
| [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk) | Python | 官方支持 |

### 2.3 MCP 服务器配置

配置示例（mcporter.json）：

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["/path/to/my-mcp-server/dist/index.js"],
      "env": {
        "API_KEY": "your-api-key"
      }
    }
  }
}
```

---

## 三、实践篇

### 3.1 从零开发一个 MCP 服务器

我们将开发一个简单的 **"待办事项" MCP 服务器**。

#### 步骤 1：项目初始化

```bash
mkdir todo-mcp-server
cd todo-mcp-server
npm init -y
npm install @modelcontextprotocol/sdk zod
npm install -D typescript @types/node
npx tsc --init
```

#### 步骤 2：定义工具

```typescript
// src/index.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";
import { z } from "zod";

// 内存存储
const todos: Array<{
  id: string;
  task: string;
  priority: string;
  completed: boolean;
}> = [];

// 创建服务器
const server = new Server(
  { name: "todo-mcp-server", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

// 工具列表
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "add_todo",
      description: "添加一个新的待办事项",
      inputSchema: {
        type: "object",
        properties: {
          task: { type: "string", description: "待办事项内容" },
          priority: { type: "string", enum: ["low", "medium", "high"] },
        },
        required: ["task"],
      },
    },
    {
      name: "list_todos",
      description: "列出所有待办事项",
      inputSchema: { type: "object", properties: {} },
    },
  ],
}));

// 工具调用
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;
  
  if (name === "add_todo") {
    const todo = {
      id: Math.random().toString(36).substring(7),
      task: String(args.task),
      priority: String(args.priority || "medium"),
      completed: false,
    };
    todos.push(todo);
    return { content: [{ type: "text", text: `✅ 已添加：${todo.task}` }] };
  }
  
  if (name === "list_todos") {
    const list = todos.map(t => `○ ${t.task}`).join("\n");
    return { content: [{ type: "text", text: list || "暂无待办事项" }] };
  }
  
  throw new Error(`Unknown tool: ${name}`);
});

// 启动
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
}

main().catch(console.error);
```

---

## 四、进阶篇

### 4.1 高级特性

#### 资源（Resources）

```typescript
server.setRequestHandler(ListResourcesRequestSchema, async () => ({
  resources: [
    {
      uri: "todos://stats",
      name: "待办统计",
      mimeType: "application/json",
    },
  ],
}));
```

#### 提示模板（Prompts）

```typescript
server.setRequestHandler(ListPromptsRequestSchema, async () => ({
  prompts: [
    {
      name: "todo_summary",
      description: "生成待办事项摘要",
    },
  ],
}));
```

### 4.2 性能优化

- 异步处理长时间任务
- 使用连接池管理数据库连接
- 添加缓存减少重复计算
- 设置合理的超时时间

### 4.3 安全考虑

- 输入验证（使用 Zod）
- 参数化查询防止注入
- 敏感信息保护
- 权限控制

---

## 五、案例篇

### 5.1 优秀项目分析

#### 案例 1：fastmcp（TypeScript 框架）

```typescript
import { FastMCP } from "fastmcp";

const mcp = new FastMCP("My Server");

mcp.addTool("calculate", {
  parameters: z.object({ expression: z.string() }),
  async execute({ expression }) {
    return String(eval(expression));
  },
});

mcp.start();
```

#### 案例 2：fetch-mcp（HTTP 工具）

支持各种 HTTP 方法，灵活配置，详细错误处理。

#### 案例 3：mysql_mcp_server（数据库）

安全查询、连接池管理、多种输出格式。

#### 案例 4：microsoft/mcp（企业级）

Azure AD 集成、审计日志、速率限制。

#### 案例 5：kubernetes-mcp-server

K8s 集群管理、RBAC 集成、命名空间隔离。

### 5.2 最佳实践

1. **单一职责**：每个 MCP 服务器专注于一个领域
2. **清晰命名**：工具和资源名称要自描述
3. **完善文档**：每个工具都要有详细描述
4. **错误友好**：提供清晰的错误信息
5. **渐进增强**：支持可选参数，提供默认值

---

## 六、资源汇总

### 6.1 官方资源

| 资源 | 链接 |
|------|------|
| 官方网站 | https://modelcontextprotocol.io |
| 官方 SDK | https://github.com/modelcontextprotocol/servers |
| Python SDK | https://github.com/modelcontextprotocol/python-sdk |
| TypeScript SDK | https://github.com/modelcontextprotocol/typescript-sdk |

### 6.2 开发工具

| 工具 | 链接 |
|------|------|
| fastmcp | https://github.com/punkpeye/fastmcp |
| mcp-framework | https://github.com/QuantGeekDev/mcp-framework |
| awesome-mcp-servers | https://github.com/wong2/awesome-mcp-servers |

### 6.3 信源统计

| 类别 | 数量 |
|------|------|
| 官方资源 | 5 |
| GitHub 项目 | 15 |
| 开发框架 | 4 |
| 精选列表 | 3 |
| 学习资源 | 5 |
| **总计** | **30** |

---

## 结语

MCP 正在重塑 AI 与外部世界的连接方式。通过标准化协议，开发者可以：

- 🚀 **快速集成** - 一次开发，多处使用
- 🔒 **安全可控** - 内置安全机制和权限管理
- 🛠️ **专注业务** - 无需关心底层通信细节
- 🌐 **开放生态** - 与全球开发者共享工具

**开始你的 MCP 开发之旅吧！**

---

*文档版本：1.0*  
*最后更新：2026年2月16日*  
*维护者：OpenClaw Community*
