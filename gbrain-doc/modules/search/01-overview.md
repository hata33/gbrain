# search — 概述

> GBrain 的混合搜索系统将向量搜索、全文搜索和知识图谱信号融合，
> 通过 RRF（Reciprocal Rank Fusion）算法产生最终排序。

## 设计思想

传统 RAG 只做向量搜索或关键词搜索，GBrain 将三种信号融合：

```
用户查询 "Alice 上季度投了什么？"
    │
    ├── 向量搜索 → 语义相近的 chunk
    ├── 全文搜索 → 关键词匹配的页面
    └── 图谱信号 → Alice 的关系网络
    │
    ▼
  RRF 融合 → 归一化 → 加权 → 重排序 → 结果
```

## 搜索管线

```
查询输入
  │
  ▼
┌──────────────────┐
│ 查询意图分类      │ ← query-intent.ts
│ (意图识别 + 模糊) │
└────────┬─────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌────────┐ ┌────────┐
│ 向量搜索│ │关键词搜索│
│ vector  │ │keyword │
└────┬───┘ └────┬───┘
     │          │
     └────┬─────┘
          ▼
    ┌──────────┐
    │ RRF 融合  │ ← reciprocal rank fusion
    └────┬─────┘
         ▼
    ┌──────────┐
    │ 归一化    │
    └────┬─────┘
         ▼
    ┌──────────┐
    │ 图谱信号  │ ← graph-signals.ts（关系密度加权）
    └────┬─────┘
         ▼
    ┌──────────┐
    │ 时间衰减  │ ← recency-decay.ts
    └────┬─────┘
         ▼
    ┌──────────┐
    │ 重排序    │ ← rerank.ts（可选 LLM reranker）
    └────┬─────┘
         ▼
    ┌──────────┐
    │ 去重      │ ← dedup.ts
    └────┬─────┘
         ▼
    ┌──────────┐
    │ Token 预算│ ← token-budget.ts（截断到预算内）
    └──────────┘
```

## RRF 融合算法

```typescript
// Reciprocal Rank Fusion
RRF_score = sum(1 / (k + rank_in_list))
// k = 60（默认参数）
// compiled_truth boost: 2.0x（综合真相版本额外加权）
// cosine re-score: blend 0.7*rrf + 0.3*cosine
```

## 搜索模式

| 模式 | 说明 | 适用场景 |
|------|------|----------|
| `hybrid` | 向量 + 全文 + 图谱 | 默认模式 |
| `vector` | 纯向量搜索 | 语义查询 |
| `keyword` | 纯关键词搜索 | 精确匹配 |
| `graph` | 图谱遍历 | 关系查询 |

## 查询意图分类

`query-intent.ts` 自动识别查询意图：

```
"Alice 的邮箱"     → person-contact（人物信息查询）
"上周开会讨论了什么" → temporal（时间相关查询）
"MRR 增长趋势"    → metric（指标查询）
"谁在 Acme 工作"  → entity-relationship（关系查询）
```

意图影响搜索管线中的权重分配。
