# facts — 事实系统

> GBrain 的事实系统从内容中提取结构化的事实声明（指标、事件、关系），
> 支持时间线追踪、质量校准和轨迹分析。

## 文件结构

| 文件 | 说明 |
|------|------|
| [README.md](README.md) | 本文件 |
| [01-overview.md](01-overview.md) | 事实系统架构 |

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/facts/extract.ts` | 事实提取 |
| `src/core/facts/extract-from-fence.ts` | 从事实围栏提取 |
| `src/core/facts/classify.ts` | 事实分类 |
| `src/core/facts/decay.ts` | 事实衰减（时效性） |
| `src/core/facts/eligibility.ts` | 资格判断 |
| `src/core/facts/fence-write.ts` | 事实围栏写入 |
| `src/core/facts/forget.ts` | 事实遗忘 |
| `src/core/facts/queue.ts` | 事实处理队列 |

## 概述

事实是 GBrain 中结构化的知识单元，比原始页面内容更精确：

### 事实类型

| 类型 | 示例 |
|------|------|
| metric | MRR: $50,000/月 |
| event | 2026-03-15 签约 Acme |
| relationship | Alice 是 Acme 的工程负责人 |
| opinion | "我认为 RAG 是未来" — Bob |

### 事实围栏（Facts Fence）

页面中的 `## Facts` 部分是结构化的事实围栏：

```markdown
## Facts
| metric | value | unit | period | source |
|--------|-------|------|--------|--------|
| mrr | 50000 | USD | monthly | 2026-03-15 meeting |
| employees | 25 | count | - | 2026-04-01 email |
```

### 事实提取流程

```
内容摄入
  → 事实提取（extract.ts）
    → LLM 提取事实声明
    → 结构化为 { metric, value, unit, period, source }
  → 事实分类（classify.ts）
    → 指标类 / 事件类 / 关系类 / 观点类
  → 事实去重
    → 同一指标的新值覆盖旧值
    → 或保留历史（取决于配置）
  → 写入 facts 表
  → 更新页面的 Facts 围栏
```

### 事实衰减

`decay.ts` — 事实随时间衰减：
- 最近的事实权重更高
- 过期的事实降低可信度
- 用于搜索排序和综合生成

### 事实遗忘

`forget.ts` — 主动遗忘过时或错误的事实：
- 被新事实覆盖的旧事实
- 明确标记为错误的事实
- 长期未被引用的事实
