# ai — 概述

> AI Gateway 是 GBrain 中所有 AI 调用的统一入口，封装了 Provider 解析、重试、错误标准化和维度管理。

## 设计思想

GBrain 的每个核心功能都依赖 AI：
- **搜索** → 向量嵌入
- **提取** → LLM 实体/事实提取
- **综合** → LLM 答案生成
- **扩展** → 查询扩展
- **转录** → 语音转文字

AI Gateway 将这些调用统一管理：

```
┌─────────────────────────────────────┐
│           AI Gateway                │
│  configureGateway(config)           │
│                                     │
│  embed(texts)    → 嵌入向量         │
│  expand(query)   → 查询扩展         │
│  generate(prompt) → LLM 生成        │
│  transcribe(audio) → 语音转录       │
└──────────────────┬──────────────────┘
                   │
     ┌─────────────┼─────────────┐
     │             │             │
     ▼             ▼             ▼
  OpenAI      Anthropic      Google
  OpenRouter   本地模型      自定义
```

## Provider 支持

| Provider | SDK | 模型 |
|----------|-----|------|
| OpenAI | `@ai-sdk/openai` | GPT-4o, text-embedding-3-* |
| Anthropic | `@ai-sdk/anthropic` | Claude Sonnet, Haiku |
| Google | `@ai-sdk/google` | Gemini Pro, Gemini Flash |
| OpenRouter | `@ai-sdk/openai-compatible` | 数百种模型 |
| 本地模型 | `@ai-sdk/openai-compatible` | Ollama, LM Studio |

## Vercel AI SDK

底层使用 Vercel AI SDK（`ai` 包），提供：
- 统一的 Provider 接口
- 流式生成
- 结构化输出（`generateObject`）
- 嵌入（`embed`, `embedMany`）

## 配置

```yaml
ai:
  provider: "openai"
  apiKey: "${OPENAI_API_KEY}"
  embeddingModel: "text-embedding-3-small"
  embeddingDimensions: 1536
  # 或使用 OpenRouter
  provider: "openrouter"
  apiKey: "${OPENROUTER_API_KEY}"
```
