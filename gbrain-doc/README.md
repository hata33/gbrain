# GBrain 文档

> 本目录包含 GBrain 项目的核心模块文档，基于源码分析生成。

## 模块索引

### P0 — 核心模块（10 个）

| 模块 | 说明 | 核心代码 |
|------|------|----------|
| [engine](modules/engine/) | 核心脑引擎 | `src/core/engine.ts` |
| [database](modules/database/) | 数据库引擎（PGLite/Postgres） | `src/core/db.ts`, `pglite-engine.ts` |
| [search](modules/search/) | 混合搜索（RRF 融合） | `src/core/search/` |
| [ai](modules/ai/) | AI Gateway（多 Provider） | `src/core/ai/` |
| [ingestion](modules/ingestion/) | 内容摄入管线 | `src/core/ingestion/` |
| [embedding](modules/embedding/) | 向量嵌入 | `src/core/embedding.ts` |
| [entities](modules/entities/) | 实体识别与管理 | `src/core/entities/` |
| [facts](modules/facts/) | 事实系统 | `src/core/facts/` |
| [extract](modules/extract/) | 内容提取 | `src/core/extract/` |
| [config](modules/config/) | 配置系统 | `src/core/config.ts` |

### P1 — 重要模块（10 个）

| 模块 | 说明 | 核心代码 |
|------|------|----------|
| [enrichment](modules/enrichment/) | 内容富化 | `src/core/enrichment-service.ts` |
| [recall](modules/recall/) | 记忆召回与综合 | `src/commands/recall.ts` |
| [brain-registry](modules/brain-registry/) | 多脑管理 | `src/core/brain-registry.ts` |
| [sources](modules/sources/) | 数据源管理 | `src/core/sources-*.ts` |
| [mcp](modules/mcp/) | MCP 服务器 | `src/mcp/` |
| [cli](modules/cli/) | CLI 接口（100+ 命令） | `src/cli.ts`, `src/commands/` |
| [knowledge-graph](modules/knowledge-graph/) | 知识图谱 | `src/core/link-extraction.ts` |
| [minions](modules/minions/) | 后台工人（任务队列） | `src/core/minions/` |
| [cycle](modules/cycle/) | 梦境循环（9 阶段维护） | `src/core/cycle.ts` |
| [think](modules/think/) | 深度推理 | `src/core/think/` |

### P2 — 辅助模块（10 个）

| 模块 | 说明 | 核心代码 |
|------|------|----------|
| [takes](modules/takes/) | 观点提取与追踪 | `src/core/takes-fence.ts` |
| [calibration](modules/calibration/) | 质量校准 | `src/core/calibration/` |
| [budget](modules/budget/) | 预算管理 | `src/core/budget/` |
| [skillpack](modules/skillpack/) | 技能包系统 | `src/core/skillpack/` |
| [transcription](modules/transcription/) | 语音转录 | `src/core/transcription.ts` |
| [code-intel](modules/code-intel/) | 代码智能 | `src/core/code-intel/` |
| [remediation](modules/remediation/) | 自我修复 | `src/core/remediation/` |
| [storage](modules/storage/) | 存储分层 | `src/core/storage/` |
| [audit](modules/audit/) | 审计系统 | `src/core/audit/` |
| [eval](modules/eval/) | 评估框架 | `src/core/eval/` |

## 项目核心特点

1. **知识大脑** — 不是简单的笔记工具，而是有综合、图谱、差距分析能力的 AI 大脑
2. **零 LLM 图谱** — 知识图谱边通过规则提取，不调用 LLM，零成本
3. **混合搜索** — 向量 + 全文 + 图谱信号三路融合（RRF）
4. **梦境循环** — 9 阶段自动化维护，"你睡觉时大脑在工作"
5. **多脑联邦** — 一个用户可以有多个大脑，支持团队共享
6. **结构化知识** — Facts（客观事实）+ Takes（主观判断）双轨知识体系
