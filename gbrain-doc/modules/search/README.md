# search — 混合搜索

> GBrain 的搜索系统实现向量搜索 + 全文搜索 + 图谱信号的混合融合（RRF），
> 是大脑检索能力的核心。

## 文件结构

| 文件 | 说明 |
|------|------|
| [README.md](README.md) | 本文件 |
| [01-overview.md](01-overview.md) | 搜索架构与管线 |
| [02-lifecycle.md](02-lifecycle.md) | 查询处理流程 |
| [03-capabilities.md](03-capabilities.md) | 搜索模式与配置 |
| [04-policies.md](04-policies.md) | 排序策略与边界 |

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/search/hybrid.ts` | 混合搜索主入口（RRF 融合） |
| `src/core/search/vector.ts` | 向量搜索 |
| `src/core/search/keyword.ts` | 全文关键词搜索 |
| `src/core/search/rerank.ts` | 重排序 |
| `src/core/search/expansion.ts` | 查询扩展 |
| `src/core/search/query-intent.ts` | 查询意图分类 |
| `src/core/search/intent-weights.ts` | 意图权重 |
| `src/core/search/two-pass.ts` | 两轮搜索（锚点扩展） |
| `src/core/search/dedup.ts` | 结果去重 |
| `src/core/search/source-boost.ts` | 数据源加权 |
| `src/core/search/recency-decay.ts` | 时间衰减 |
| `src/core/search/graph-signals.ts` | 图谱信号 |
| `src/core/search/token-budget.ts` | Token 预算 |
| `src/core/search/mode.ts` | 搜索模式切换 |
| `src/core/search/query-cache.ts` | 查询缓存 |
