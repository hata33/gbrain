# ai — AI/LLM 集成

> GBrain 的 AI Gateway 统一管理所有 AI 调用：嵌入、生成、扩展、转录等，
> 支持 OpenAI、Anthropic、Google、OpenRouter 等多 Provider。

## 文件结构

| 文件 | 说明 |
|------|------|
| [README.md](README.md) | 本文件 |
| [01-overview.md](01-overview.md) | AI Gateway 架构 |
| [02-lifecycle.md](02-lifecycle.md) | 配置、初始化、调用流程 |
| [03-capabilities.md](03-capabilities.md) | 能力清单 |
| [04-policies.md](04-policies.md) | Provider 选择、错误处理 |

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/ai/gateway.ts` | AI Gateway 核心（统一入口） |
| `src/core/ai/model-resolver.ts` | 模型解析与选择 |
| `src/core/ai/capabilities.ts` | 模型能力检测 |
| `src/core/ai/defaults.ts` | 默认配置（模型、维度等） |
| `src/core/ai/probes.ts` | Provider 连接探测 |
| `src/core/ai/errors.ts` | AI 错误分类 |
| `src/core/ai/recipes/` | 预设配方 |
| `src/core/embedding.ts` | 嵌入服务（委托给 Gateway） |
