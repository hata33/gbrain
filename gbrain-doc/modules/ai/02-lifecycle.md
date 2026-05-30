# ai — 生命周期

> AI Gateway 从配置到调用的完整流程。

## 初始化流程

```
CLI 启动 / MCP Server 启动
  │
  ▼
configureGateway(config)
  → 读取 AI 配置（provider, apiKey, model...）
  → 创建 Provider 客户端实例
  → 按 (provider, modelId, baseUrl) 缓存
  → 探测 Provider 可用性（probes.ts）
  │
  ▼
Gateway 就绪，接受调用
```

## 嵌入调用流程

```
embed(["text1", "text2"])
  → 解析嵌入模型（model-resolver.ts）
  → 检查维度配置（默认 1536）
  → 调用 Vercel AI SDK embedMany()
  → 重试（指数退避）
  → 错误标准化（errors.ts）
  → 返回向量数组
```

## LLM 生成流程

```
generateText({ prompt, model })
  → 解析生成模型
  → 构建消息格式
  → 调用 Vercel AI SDK generateText()
  → 重试（指数退避）
  → 错误标准化
  → 返回文本结果
```

## 多模态嵌入

```
embedMultimodal({ text, imageUrl })
  → 文本嵌入 → text_embedding 向量
  → 图片嵌入 → image_embedding 向量
  → 存储到不同列（embedding_text, embedding_image）
```

## 错误分类

```typescript
// src/core/ai/errors.ts
AIConfigError    // 配置错误（API Key 无效、模型不存在）→ 不重试
AITransientError // 瞬时错误（超时、限流）→ 可重试
```
