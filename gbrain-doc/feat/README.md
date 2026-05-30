# GBrain 核心特性实现原理

> 本目录基于 GBrain 项目实际代码，深入分析每个核心特性的实现原理。

## 特性索引

| 文档 | 特性 | 核心代码位置 |
|------|------|-------------|
| [01-hybrid-search.md](01-hybrid-search.md) | 混合搜索（RRF 融合） | `src/core/search/` |
| [02-zero-llm-knowledge-graph.md](02-zero-llm-knowledge-graph.md) | 零 LLM 知识图谱 | `src/core/link-extraction.ts` |
| [03-dream-cycle.md](03-dream-cycle.md) | 梦境循环（9 阶段维护） | `src/core/cycle.ts` |
| [04-structured-knowledge.md](04-structured-knowledge.md) | 结构化知识（Facts + Takes） | `src/core/facts/`, `src/core/takes-fence.ts` |
| [05-synthesis-gap-analysis.md](05-synthesis-gap-analysis.md) | 综合与差距分析 | `src/core/think/` |
| [06-multi-brain-federation.md](06-multi-brain-federation.md) | 多脑联邦 | `src/core/brain-registry.ts` |
| [07-self-healing.md](07-self-healing.md) | 自我修复（Doctor 系统） | `src/core/remediation/` |

## 项目核心特点

1. **知识大脑** — 不是搜索工具，而是有综合、图谱、差距分析能力的 AI 大脑
2. **零 LLM 图谱** — 知识图谱边通过规则提取，零 API 成本
3. **混合搜索** — 向量 + 全文 + 图谱三路融合，Benchmark P@5 49.1%
4. **梦境循环** — 9 阶段自动化维护，"你睡觉时大脑在工作"
5. **多脑联邦** — 一个用户可以有多个大脑，支持团队 RLS 共享
6. **结构化知识** — Facts（客观事实）+ Takes（主观判断）双轨体系
7. **综合回答** — 不是返回搜索结果，而是生成带引用的答案 + 差距分析
8. **自我修复** — Doctor 系统自动检测和修复健康问题
