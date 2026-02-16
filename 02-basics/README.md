# 基础篇：协议与环境

## MCP 协议规范

### 传输方式

| 方式 | 适用场景 |
|------|----------|
| **stdio** | 本地进程通信 |
| **SSE** | 远程服务器 |

### 消息格式

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "search",
    "arguments": { "query": "MCP" }
  }
}
```

## 开发环境搭建

### TypeScript

```bash
npm install @modelcontextprotocol/sdk zod
```

### Python

```bash
pip install mcp
```

## 配置示例

```json
{
  "mcpServers": {
    "my-server": {
      "command": "node",
      "args": ["dist/index.js"],
      "env": { "API_KEY": "xxx" }
    }
  }
}
```

---

[下一章：实践篇 →](../03-practice/README.md)
