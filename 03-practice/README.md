# 实践篇：开发 MCP 服务器

## 完整示例：待办事项服务器

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { CallToolRequestSchema, ListToolsRequestSchema } from "@modelcontextprotocol/sdk/types.js";

const todos: any[] = [];

const server = new Server(
  { name: "todo-server", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "add_todo",
      description: "添加待办事项",
      inputSchema: {
        type: "object",
        properties: { task: { type: "string" } },
        required: ["task"],
      },
    },
    {
      name: "list_todos",
      description: "列出待办事项",
      inputSchema: { type: "object", properties: {} },
    },
  ],
}));

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;
  
  if (name === "add_todo") {
    todos.push({ id: Date.now().toString(), task: args.task, done: false });
    return { content: [{ type: "text", text: `✅ 已添加：${args.task}` }] };
  }
  
  if (name === "list_todos") {
    return { content: [{ type: "text", text: todos.map(t => `○ ${t.task}`).join("\n") || "暂无" }] };
  }
  
  throw new Error(`Unknown: ${name}`);
});

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
}

main();
```

## 测试

```bash
npx tsc
node dist/index.js
```

---

[下一章：进阶篇 →](../04-advanced/README.md)
