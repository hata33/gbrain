# ai — 策略与边界

> AI Gateway 的 Provider 选择策略、错误处理和边界情况。

## Provider 选择策略

```
配置指定 provider
  → 直接使用
  → 不指定 → 根据 API Key 自动检测
    → OPENAI_API_KEY → OpenAI
    → ANTHROPIC_API_KEY → Anthropic
    → GOOGLE_API_KEY → Google
    → 都没有 → 报错
```

## 重试策略

```
AI 调用失败
  → AITransientError（超时、429 限流、500 服务器错误）
    → 指数退避重试（最多 3 次）
    → 间隔：1s → 2s → 4s
  → AIConfigError（API Key 无效、模型不存在）
    → 不重试，立即报错
```

## 维度兼容

- 嵌入维度在创建时固定，不可更改
- 更换嵌入模型需要重新嵌入所有 chunk（`gbrain reindex`）
- `embedding-dim-check.ts` 启动时验证维度一致性

## 环境变量

| 变量 | 说明 |
|------|------|
| `OPENAI_API_KEY` | OpenAI API Key |
| `ANTHROPIC_API_KEY` | Anthropic API Key |
| `GOOGLE_API_KEY` | Google AI API Key |
| `OPENROUTER_API_KEY` | OpenRouter API Key |
| `GBRAIN_EMBEDDING_MODEL` | 覆盖默认嵌入模型 |
| `GBRAIN_EMBEDDING_DIMENSIONS` | 覆盖默认嵌入维度 |

## 边界情况

1. **无 API Key** → `isAvailable()` 返回 false，跳过 AI 相关功能
2. **模型不可用** → fallback 到备用模型（如配置了的话）
3. **嵌入维度不匹配** → 启动时报错，提示重新嵌入
4. **Provider 限流** → 自动重试，超过最大重试后报错
5. **长文本嵌入** → 自动截断到模型最大 token 限制
