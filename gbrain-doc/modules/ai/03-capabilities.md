# ai — 能力清单

> AI Gateway 的完整能力清单。

## 核心 API

| 函数 | 说明 |
|------|------|
| `configureGateway(config)` | 初始化 AI Gateway |
| `embed(texts)` | 批量文本嵌入 |
| `embedOne(text)` | 单条文本嵌入 |
| `embedQuery(text)` | 查询嵌入（可能使用不同模型） |
| `embedMultimodal(input)` | 多模态嵌入（文本+图片） |
| `expand(query)` | 查询扩展（生成同义变体） |
| `generateText(params)` | LLM 文本生成 |
| `generateObject(params)` | LLM 结构化输出 |
| `isAvailable(touchpoint)` | 检查 AI 是否可用 |
| `getEmbeddingDimensions()` | 获取嵌入维度 |
| `getEmbeddingModel()` | 获取嵌入模型名 |

## 模型解析

```typescript
// model-resolver.ts
resolveModel(config)
  → 解析 provider + model 组合
  → 支持别名（如 "haiku" → "claude-3-haiku"）
  → 支持 fallback（主模型不可用时使用备用）
```

## 模型能力检测

```typescript
// capabilities.ts
detectCapabilities(model)
  → 是否支持图片输入
  → 是否支持函数调用
  → 最大上下文窗口
  → 是否支持流式输出
```

## 预设配方

```typescript
// recipes/
listRecipes()
  → 返回预配置的模型组合
  → 如：嵌入用 text-embedding-3-small，生成用 gpt-4o
```

## 维度管理

| 模型 | 默认维度 |
|------|----------|
| text-embedding-3-small | 1536 |
| text-embedding-3-large | 3072 |
| text-embedding-ada-002 | 1536 |

维度在 `initSchema()` 时确定，后续不可更改（需要重新嵌入所有 chunk）。
