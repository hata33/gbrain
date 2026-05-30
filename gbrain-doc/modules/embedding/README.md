# embedding — 向量嵌入

> GBrain 的嵌入服务将文本和图片转换为向量表示，是搜索和语义理解的基础。

## 文件结构

| 文件 | 说明 |
|------|------|
| [README.md](README.md) | 本文件 |
| [01-overview.md](01-overview.md) | 嵌入架构与模型 |

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/embedding.ts` | 嵌入服务入口（委托给 AI Gateway） |
| `src/core/embedding-context.ts` | 嵌入上下文管理 |
| `src/core/embed-preflight.ts` | 嵌入预检 |
| `src/core/embed-skip.ts` | 跳过嵌入的条件 |
| `src/core/embed-stale.ts` | 过期嵌入检测 |
| `src/core/embedding-dim-check.ts` | 维度一致性检查 |
| `src/core/embedding-pricing.ts` | 嵌入成本估算 |

## 概述

嵌入是 GBrain 搜索能力的基础。每个内容块（chunk）被转换为向量，存储在 pgvector 列中，支持余弦相似度搜索。

### 嵌入流程

```
文本内容
  → 切块（chunkers/）
  → embed(chunks) → AI Gateway → Provider API
  → 返回向量数组
  → 存储到 content_chunks.embedding 列
```

### 多模态嵌入

```
图片内容
  → embedMultimodal({ imageUrl })
  → 返回 image_embedding 向量
  → 存储到 content_chunks.embedding_image 列
```

### 向量索引

| 引擎 | 索引类型 | 说明 |
|------|----------|------|
| PGLite | 无索引 | 全表扫描（小数据集足够） |
| Postgres | IVFFlat / HNSW | 向量索引（大数据集） |

### 维度配置

默认维度 1536（text-embedding-3-small），可在配置中覆盖。维度在创建时固定，更改需要重新嵌入所有 chunk。
