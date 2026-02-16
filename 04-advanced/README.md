# 进阶篇：高级特性

## Resources（资源）

```typescript
server.setRequestHandler(ListResourcesRequestSchema, async () => ({
  resources: [{ uri: "data://stats", name: "统计数据", mimeType: "application/json" }],
}));

server.setRequestHandler(ReadResourceRequestSchema, async (request) => ({
  contents: [{ uri: request.params.uri, mimeType: "application/json", text: JSON.stringify({ count: 10 }) }],
}));
```

## Prompts（提示模板）

```typescript
server.setRequestHandler(ListPromptsRequestSchema, async () => ({
  prompts: [{ name: "summary", description: "生成摘要" }],
}));
```

## 性能优化

- ✅ 异步处理
- ✅ 连接池
- ✅ 缓存
- ✅ 超时控制

## 安全考虑

- ✅ 输入验证
- ✅ 参数化查询
- ✅ 权限控制
- ✅ 日志脱敏

---

[下一章：案例篇 →](../05-cases/README.md)
