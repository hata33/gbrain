# 01 — 混合搜索（RRF 融合）

> GBrain 的搜索系统融合向量搜索、全文搜索和知识图谱信号，
> 通过 RRF（Reciprocal Rank Fusion）算法产生最终排序，是"大脑 vs 搜索引擎"的核心差异。

## 问题

传统 RAG 只做向量搜索或关键词搜索：
- **纯向量搜索** — 语义理解强，但精确匹配弱
- **纯关键词搜索** — 精确匹配强，但语义理解弱
- **两者都不利用知识图谱** — 无法回答关系查询

## 解决方案

GBrain 将三种信号融合：

```
查询 "Alice 上季度投了什么？"
    │
    ├── 向量搜索 → 语义相近的 chunk（Alice 相关页面）
    ├── 全文搜索 → 关键词匹配（"Alice"、"投资"）
    └── 图谱信号 → Alice 的关系网络（works_at, invested_in 边）
    │
    ▼
  RRF 融合 → 归一化 → 加权 → 重排序 → 最终结果
```

## RRF 算法

```
RRF_score = Σ 1 / (k + rank)
k = 60

加权层：
  → compiled_truth boost: 2.0x
  → cosine re-score: blend 0.7*rrf + 0.3*cosine
  → 图谱信号：关系密度加权
  → 时间衰减：最近的内容优先
  → 源加权：不同数据源可配置不同权重
```

## 搜索管线

```
查询 → 意图分类 → 查询扩展 → 并行搜索（向量+全文）
  → RRF 融合 → 图谱加权 → 时间衰减 → 源加权
  → 重排序（可选 LLM reranker） → 去重 → Token 预算截断 → 结果
```

## 关键设计

1. **意图感知** — 查询意图影响权重分配（temporal → 时间衰减增强，entity-relationship → 图谱增强）
2. **两轮搜索** — 复杂查询先找锚点，再扩展上下文
3. **查询缓存** — 规范化查询缓存，避免重复计算
4. **源范围控制** — 可限制在特定数据源内搜索

## Benchmark

在 240 页 Opus 生成语料库上：
- **P@5: 49.1%**，**R@5: 97.9%**
- 比无图谱变体高 **+31.4 个百分点 P@5**
- 比纯 ripgrep-BM25 + 向量 RAG 高类似幅度

## 核心代码

| 文件 | 职责 |
|------|------|
| `src/core/search/hybrid.ts` | RRF 融合主入口 |
| `src/core/search/query-intent.ts` | 查询意图分类 |
| `src/core/search/graph-signals.ts` | 图谱信号加权 |
| `src/core/search/two-pass.ts` | 两轮搜索 |
