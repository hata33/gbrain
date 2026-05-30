# search — 能力清单

> 搜索系统的完整能力清单，包括搜索模式、配置选项和集成点。

## 搜索模式

| 模式 | 引擎 | 说明 |
|------|------|------|
| `hybrid` | 向量 + 全文 + 图谱 | 默认模式，三种信号融合 |
| `vector` | 仅向量 | 纯语义搜索 |
| `keyword` | 仅全文 | pg_trgm 关键词匹配 |
| `graph` | 图遍历 | 关系网络查询 |

## 查询意图类型

| 意图 | 触发条件 | 权重调整 |
|------|----------|----------|
| `person-contact` | 人名 + 联系方式 | 实体 boost |
| `temporal` | 时间词（上周、上月） | 时间衰减增强 |
| `metric` | 指标词（MRR、收入） | 事实 boost |
| `entity-relationship` | 关系词（谁、在哪工作） | 图谱 boost |
| `code` | 代码相关词 | 代码文件 boost |

## 配置选项

```yaml
search:
  mode: "hybrid"              # 搜索模式
  reranker: true              # 启用 LLM 重排序
  expansion: true             # 启用查询扩展
  twoPass: true               # 启用两轮搜索
  maxResults: 20              # 最大结果数
  minScore: 0.1               # 最低分数阈值
  recencyDecay: true          # 启用时间衰减
  graphSignals: true          # 启用图谱信号
  tokenBudget: 8000           # Token 预算
  queryCache: true            # 启用查询缓存
```

## RRF 参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `k` | 60 | RRF 常数（越大，排名靠后的结果权重越高） |
| `cosineBlend` | 0.3 | 余弦相似度混合比例 |
| `compiledTruthBoost` | 2.0 | 综合真相版本加权倍数 |

## 源范围控制

```typescript
search(query, {
  sourceId: 'work',           // 限制在单个源
  sourceIds: ['work', 'code'], // 限制在多个源
  // 不指定 → 搜索所有源
})
```

## 遥测

`telemetry.ts` 记录每次搜索的性能指标：
- 查询耗时
- 向量搜索耗时
- 全文搜索耗时
- 融合耗时
- 结果数量
- 缓存命中率
