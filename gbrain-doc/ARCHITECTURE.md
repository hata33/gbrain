# GBrain 项目架构

> 本文档描述 GBrain 项目的整体架构设计、分层结构、各层职责与核心模块组成。

## 一、项目概览

GBrain 是一个 **知识引擎 / 记忆脑系统**，能够从多种数据源摄入内容，通过 AI 进行理解、提取、关联，构建可搜索的知识图谱。它支持本地运行（PGLite）和远程部署（PostgreSQL），并提供 MCP 协议对外暴露能力。

**技术栈：**

| 项目 | 技术 |
|------|------|
| 语言 | TypeScript |
| 运行时 | Bun |
| 数据库 | PGLite (本地) / PostgreSQL (远程) |
| 向量搜索 | pgvector |
| AI 接口 | 多 Provider (OpenAI, Anthropic, Google 等) |
| 协议 | MCP (Model Context Protocol) |
| 配置 | YAML (gbrain.yml) |

---

## 二、架构分层

GBrain 采用 **八层架构**，从上到下依次为：

```
┌─────────────────────────────────────────────────────────────┐
│                    接入层 (Access Layer)                      │
│          CLI 命令 │ MCP Server │ HTTP API │ OpenClaw 插件     │
└────────────────────────┬────────────────────────────────────┘
                         │
         ┌───────────────┴───────────────┐
         │       命令层 (Command Layer)    │
         │    114 个 CLI 命令              │
         │    src/commands/               │
         └───────────────┬───────────────┘
                         │
         ┌───────────────┴───────────────┐
         │     认知引擎层 (Cognitive)      │
         │  上下文引擎 │ 思维链 │ 推理     │
         │  src/core/context-engine.ts    │
         │  src/core/think/              │
         └───────────────┬───────────────┘
                         │
         ┌───────────────┴───────────────┐
         │      知识层 (Knowledge)         │
         │  实体 │ 事实 │ 搜索 │ 关联      │
         │  src/core/entities/            │
         │  src/core/facts/               │
         │  src/core/search/              │
         └───────────────┬───────────────┘
                         │
         ┌───────────────┴───────────────┐
         │      摄入层 (Ingestion)         │
         │  导入 │ 提取 │ 嵌入 │ 去重      │
         │  src/core/ingestion/           │
         │  src/core/extract/             │
         │  src/core/embedding.ts         │
         └───────────────┬───────────────┘
                         │
         ┌───────────────┴───────────────┐
         │      AI 层 (AI Gateway)        │
         │  多 Provider 调度 │ 模型路由    │
         │  src/core/ai/                  │
         │  src/core/model-config.ts      │
         └───────────────┬───────────────┘
                         │
         ┌───────────────┴───────────────┐
         │      存储层 (Storage)           │
         │  PGLite │ PostgreSQL │ 向量索引  │
         │  src/core/db.ts                │
         │  src/core/pglite-engine.ts     │
         │  src/core/postgres-engine.ts   │
         └───────────────┬───────────────┘
                         │
         ┌───────────────┴───────────────┐
         │    基础设施层 (Infrastructure)   │
         │  配置 │ 日志 │ 安全 │ 迁移      │
         │  src/core/config.ts            │
         │  src/core/migrate.ts           │
         └───────────────────────────────┘
```

---

## 三、各层详解

### 3.1 接入层（Access Layer）

**目录：** `src/cli.ts`、`src/mcp/`、`src/commands/serve.ts`、`openclaw.plugin.json`

**职责：** 提供多种方式接入 GBrain 系统。

| 接入方式 | 模块 | 说明 |
|----------|------|------|
| CLI 命令 | `src/cli.ts` | 114+ 个命令行命令 |
| MCP Server | `src/mcp/server.ts` | 通过 MCP 协议暴露能力 |
| HTTP API | `src/commands/serve-http.ts` | HTTP 服务接口 |
| OpenClaw 插件 | `openclaw.plugin.json` | 作为 OpenClaw 插件运行 |

**MCP 模块：**

| 文件 | 功能 |
|------|------|
| `server.ts` | MCP 服务端主入口 |
| `tool-defs.ts` | 工具定义（暴露给 AI 的能力） |
| `dispatch.ts` | 请求分发 |
| `http-transport.ts` | HTTP 传输层 |
| `rate-limit.ts` | 速率限制 |

---

### 3.2 命令层（Command Layer）

**目录：** `src/commands/`

**职责：** 114 个 CLI 命令，覆盖知识引擎的所有操作。

**命令分类：**

| 类别 | 示例命令 | 说明 |
|------|----------|------|
| **初始化** | `init`, `init-mode-picker`, `init-provider-picker` | 系统初始化与配置 |
| **数据导入** | `import`, `extract`, `backfill` | 从多种来源导入数据 |
| **搜索** | `search`, `recall`, `graph-query`, `backlinks` | 知识检索与查询 |
| **内容管理** | `pages`, `takes`, `sources`, `files` | 内容的增删改查 |
| **AI 能力** | `think`, `dream`, `brainstorm`, `conversation-parser` | AI 推理与分析 |
| **代码智能** | `code-def`, `code-refs`, `code-callers`, `code-callees` | 代码分析与索引 |
| **评估** | `eval`, `eval-*` (20+ 个) | 质量评估与基准测试 |
| **维护** | `doctor`, `migrate`, `sync`, `reindex` | 系统维护与修复 |
| **多脑管理** | `brain-registry`, `brain-resolver` | 多脑注册与解析 |
| **技能** | `skillify`, `skillpack` | 技能生成与管理 |

---

### 3.3 认知引擎层（Cognitive Engine）

**目录：** `src/core/context-engine.ts`、`src/core/think/`、`src/core/cycle.ts`

**职责：** 上下文管理、推理链编排、多步思维过程。

| 模块 | 文件 | 功能 |
|------|------|------|
| 上下文引擎 | `context-engine.ts` | 管理对话上下文，动态组装提示词 |
| 思维链 | `think/index.ts` | 多步推理，支持 gather → intent → cite-render 流程 |
| 循环引擎 | `cycle.ts` (89KB) | 核心处理循环，编排摄入、提取、关联等流程 |
| 路由评估 | `routing-eval.ts` | 智能路由，决定请求由哪个模块处理 |

**思维链流程：**
```
用户查询 → intent（意图识别）→ gather（信息收集）→ cite-render（引用渲染）→ 响应
```

---

### 3.4 知识层（Knowledge Layer）

**目录：** `src/core/entities/`、`src/core/facts/`、`src/core/search/`

**职责：** 知识的结构化表示、存储与检索。

| 模块 | 目录 | 功能 |
|------|------|------|
| 实体系统 | `entities/` | 实体识别、消歧、关联 |
| 事实系统 | `facts/` | 事实抽取、分类、衰减、吸收 |
| 搜索引擎 | `search/` | 混合搜索（向量 + 关键词 + RRF 融合） |
| 关联图谱 | `edges-backfill` | 知识节点间的关联关系 |

**搜索子模块：**

| 文件 | 功能 |
|------|------|
| `embedding-column.ts` | 向量列管理 |
| `expansion.ts` | 查询扩展 |
| `dedup.ts` | 结果去重 |
| `by-image.ts` | 图片搜索 |
| `eval.ts` | 搜索质量评估 |

**事实系统子模块：**

| 文件 | 功能 |
|------|------|
| `classify.ts` | 事实分类 |
| `decay.ts` | 事实衰减（时效性） |
| `eligibility.ts` | 事实适用性判断 |
| `absorb-log.ts` | 事实吸收日志 |
| `backstop.ts` | 事实兜底机制 |

---

### 3.5 摄入层（Ingestion Layer）

**目录：** `src/core/ingestion/`、`src/core/extract/`、`src/core/import-file.ts`

**职责：** 从多种数据源摄入内容，进行提取、嵌入、去重处理。

| 模块 | 文件 | 功能 |
|------|------|------|
| 摄入管线 | `ingestion/` | 数据摄入的主流程编排 |
| 去重 | `ingestion/dedup.ts` | 内容去重，避免重复摄入 |
| 守护进程 | `ingestion/daemon.ts` | 后台持续摄入 |
| 内容提取 | `extract/` | 从原始内容中提取结构化信息 |
| 文件导入 | `import-file.ts` (62KB) | 文件导入的核心逻辑 |
| 链接提取 | `link-extraction.ts` | 从内容中提取链接 |
| NER 提取 | `extract-ner.ts` | 命名实体识别 |
| 时间线提取 | `extract-timeline-from-meetings.ts` | 从会议记录提取时间线 |
| 观点提取 | `extract-takes-from-pages.ts` | 从页面提取观点 |

**支持的数据源：**
- 本地文件（Markdown、文本、代码）
- 网页 URL
- RSS 订阅
- 会议记录
- 对话历史

---

### 3.6 AI 层（AI Gateway）

**目录：** `src/core/ai/`、`src/core/model-config.ts`

**职责：** 多 AI Provider 调度、模型选择、请求路由。

| 模块 | 文件 | 功能 |
|------|------|------|
| AI 网关 | `ai/gateway.ts` | 统一的 AI 请求入口 |
| 能力声明 | `ai/capabilities.ts` | 各模型能力声明（上下文长度、多模态等） |
| 默认配置 | `ai/defaults.ts` | 默认模型配置 |
| 维度管理 | `ai/dims.ts` | 嵌入维度管理 |
| 错误处理 | `ai/errors.ts` | AI 请求错误处理 |
| 模型配置 | `model-config.ts` | 模型选择与配置管理 |
| 模型标识 | `model-id.ts` | 模型 ID 解析 |

**支持的 Provider：** OpenAI、Anthropic、Google、本地模型等。

---

### 3.7 存储层（Storage Layer）

**目录：** `src/core/db.ts`、`src/core/pglite-engine.ts`、`src/core/postgres-engine.ts`、`src/core/storage/`

**职责：** 数据持久化，支持本地和远程两种存储模式。

| 模块 | 文件 | 功能 |
|------|------|------|
| 数据库抽象 | `db.ts` | 统一的数据库接口 |
| PGLite 引擎 | `pglite-engine.ts` (224KB) | 本地嵌入式 PostgreSQL |
| PostgreSQL 引擎 | `postgres-engine.ts` (236KB) | 远程 PostgreSQL 连接 |
| 向量索引 | `vector-index.ts` | pgvector 向量索引管理 |
| Schema | `pglite-schema.ts` | 数据库 Schema 定义 |
| 迁移 | `migrate.ts` (251KB) | 数据库迁移脚本 |
| 存储抽象 | `storage/` | 本地/S3/Supabase 存储抽象 |
| 数据库锁 | `db-lock.ts` | 并发访问锁机制 |

**存储模式：**

| 模式 | 引擎 | 适用场景 |
|------|------|----------|
| 本地 | PGLite | 个人使用，零配置 |
| 远程 | PostgreSQL | 团队协作，生产环境 |
| 云存储 | S3 / Supabase | 文件存储 |

---

### 3.8 基础设施层（Infrastructure Layer）

**目录：** `src/core/config.ts`、`src/core/errors.ts`、`src/core/retry.ts` 等

**职责：** 提供配置管理、错误处理、安全校验等基础能力。

| 模块 | 文件 | 功能 |
|------|------|------|
| 配置系统 | `config.ts` (35KB) | YAML 配置加载与管理 |
| 错误处理 | `errors.ts` | 统一错误类型 |
| 重试机制 | `retry.ts` | 请求重试与退避 |
| SSRF 防护 | `ssrf-validate.ts` | SSRF 安全校验 |
| URL 安全 | `url-safety.ts` | URL 安全检查 |
| 内容审计 | `content-sanity.ts` | 内容完整性校验 |
| 进度显示 | `progress.ts` | 任务进度条 |
| 预算追踪 | `budget/budget-tracker.ts` | API 调用预算控制 |
| 特性开关 | `feature-flags.ts` | 功能开关管理 |
| 并发控制 | `sync-concurrency.ts` | 并发任务控制 |
| 超时管理 | `timeout.ts` | 请求超时控制 |
| 连接管理 | `connection-manager.ts` | 数据库连接池 |

---

## 四、数据流

### 4.1 数据摄入流程

```
数据源（文件/URL/RSS/会议记录）
  │
  ▼
导入引擎 (import-file.ts)
  │
  ▼
内容提取 (extract/)
  │  ├── NER 实体识别
  │  ├── 链接提取
  │  ├── 时间线提取
  │  └── 观点提取
  ▼
嵌入生成 (embedding.ts)
  │
  ▼
去重检查 (ingestion/dedup.ts)
  │
  ▼
存储写入 (db.ts → pglite/postgres)
  │
  ▼
知识关联 (entities + facts + search)
```

### 4.2 知识检索流程

```
用户查询
  │
  ▼
意图识别 (think/intent.ts)
  │
  ▼
混合搜索 (search/)
  │  ├── 向量相似度搜索
  │  ├── 关键词搜索
  │  └── RRF 融合排序
  ▼
上下文组装 (context-engine.ts)
  │
  ▼
AI 推理 (ai/gateway.ts)
  │
  ▼
引用渲染 (think/cite-render.ts)
  │
  ▼
响应输出
```

### 4.3 MCP 服务流程

```
AI 客户端（Claude / OpenClaw / ...）
  │
  ▼
MCP Server (mcp/server.ts)
  │
  ▼
工具分发 (mcp/dispatch.ts)
  │
  ▼
命令执行 (commands/)
  │
  ▼
结果返回
```

---

## 五、核心子系统

### 5.1 多脑管理（Brain Registry）

**模块：** `brain-registry.ts`、`brain-resolver.ts`

支持管理多个独立的"知识脑"，每个脑有独立的数据和配置。

### 5.2 评估系统（Eval）

**模块：** `src/core/eval/`、`src/commands/eval*.ts`（20+ 个评估命令）

GBrain 内置完整的评估框架，用于衡量搜索质量、提取准确度、推理能力等。

### 5.3 技能系统（Skillify）

**模块：** `src/core/skillify/`、`src/core/skillpack/`

将知识转化为可复用的技能包，支持导出和分享。

### 5.4 代码智能（Code Intelligence）

**模块：** `src/core/code-intel/`

支持代码分析，包括定义查找、引用追踪、调用图构建。

### 5.5 内容富化（Enrichment）

**模块：** `src/core/enrichment-service.ts`

自动为摄入的内容添加元数据、标签、摘要等富化信息。

---

## 六、设计原则

| 原则 | 说明 |
|------|------|
| **本地优先** | PGLite 零配置即可运行，无需外部数据库 |
| **多引擎兼容** | 存储层抽象，支持 PGLite / PostgreSQL 无缝切换 |
| **AI 原生** | 深度集成 AI，用于理解、提取、推理 |
| **可评估** | 内置 20+ 评估命令，量化系统质量 |
| **可扩展** | 通过 MCP 协议对外暴露能力，易于集成 |
| **安全防护** | SSRF 验证、URL 安全检查、内容审计 |

---

## 七、模块统计

| 类别 | 数量 | 说明 |
|------|------|------|
| 源码目录 | 36+ | `src/core/` 下的子目录 |
| CLI 命令 | 114 | `src/commands/` 下的命令文件 |
| 核心文件 | 150+ | `src/core/` 下的独立文件 |
| MCP 工具 | 10+ | 通过 MCP 暴露的工具 |
| 评估命令 | 20+ | `eval-*` 系列命令 |
| 存储引擎 | 2 | PGLite + PostgreSQL |
| AI Provider | 4+ | OpenAI, Anthropic, Google 等 |

---

## 八、相关文档

各模块详细文档请参阅 [modules/](modules/) 目录，特性分析请参阅 [feat/](feat/) 目录。
