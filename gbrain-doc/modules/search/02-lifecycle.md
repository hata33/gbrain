# search — 生命周期

> 搜索查询从输入到返回结果的完整处理流程。

## 查询处理流程

```
用户输入查询字符串
  │
  ▼
1. 查询预处理
   → 去除特殊字符
   → 检测查询意图（query-intent.ts）
   → 检测模糊模态（图片/文本混合查询）
   → 查询缓存检查（query-cache.ts）
     → 缓存命中 → 直接返回
     → 缓存未命中 → 继续
  │
  ▼
2. 查询扩展（expansion.ts）
   → LLM 扩展查询为多个变体
   → 生成同义词、相关概念
   → 用于提高召回率
  │
  ▼
3. 并行执行搜索
   ├── 向量搜索（vector.ts）
   │   → embedQuery() 生成查询向量
   │   → cosine similarity 搜索
   │   → 返回 top-K 结果
   │
   └── 全文搜索（keyword.ts）
       → pg_trgm 相似度搜索
       → ILIKE 模式匹配
       → 返回 top-K 结果
  │
  ▼
4. RRF 融合（hybrid.ts）
   → 向量结果和关键词结果合并
   → Reciprocal Rank Fusion 计算分数
   → compiled_truth boost（2.0x）
  │
  ▼
5. 后处理
   → 图谱信号加权（graph-signals.ts）
   → 时间衰减（recency-decay.ts）
   → 数据源加权（source-boost.ts）
   → 精确匹配加权（intent-weights.ts）
  │
  ▼
6. 重排序（rerank.ts，可选）
   → LLM reranker 重新排序
   → 仅在配置启用时执行
  │
  ▼
7. 两轮搜索（two-pass.ts，可选）
   → 第一轮：锚点搜索
   → 第二轮：锚点 chunk 扩展
   → 用于需要上下文的复杂查询
  │
  ▼
8. 去重（dedup.ts）
   → 按 slug 去重（同一页面只保留最高分 chunk）
   → 内容相似度去重
  │
  ▼
9. Token 预算截断（token-budget.ts）
   → 截断到 LLM 上下文窗口预算内
   → 保留最高分的结果
  │
  ▼
10. 返回结果
    → 记录搜索遥测（telemetry.ts）
    → 写入查询缓存
```

## 搜索模式切换

`mode.ts` — 根据查询特征自动选择搜索模式：

```
查询很短 + 精确 → keyword 模式
查询是自然语言 → hybrid 模式
查询包含实体名 → hybrid + graph boost
查询包含图片 → multimodal 模式
```

## 两轮搜索

`two-pass.ts` — 处理需要上下文的复杂查询：

```
第一轮：搜索锚点 chunk（最相关的片段）
  → 获取锚点所在页面的所有 chunk
  → 获取锚点的关联页面

第二轮：用锚点上下文扩展结果
  → 补充同一页面的其他 chunk
  → 补充关联页面的内容
  → 让 LLM 获得完整上下文
```
