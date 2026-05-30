# database — 概述

> GBrain 的数据库层支持两种后端：PGLite（嵌入式 WebAssembly Postgres）和 Postgres（生产级），
> 共享同一套 Schema 和 SQL 查询逻辑。

## 双引擎架构

```
BrainEngine 接口
    ├── PGLiteEngine  → @electric-sql/pglite (WASM)
    └── PostgresEngine → postgres.js (TCP 连接)
```

两种引擎实现完全相同的 BrainEngine 接口，共享相同的 SQL 语句（Postgres 方言）。

## PGLite 引擎

`src/core/pglite-engine.ts` — 基于 WebAssembly 的嵌入式 Postgres：

- **零配置**：无需安装 Postgres 服务器
- **WASM 运行时**：在 Node.js 中运行完整的 Postgres
- **扩展支持**：pgvector（向量搜索）、pg_trgm（模糊匹配）
- **本地存储**：数据文件存储在 `~/.gbrain/data/`
- **文件锁**：`pglite-lock.ts` 防止多进程并发访问
- **自定义 Schema**：`pglite-schema.ts` 包含 PGLite 特有的 Schema 调整

### 限制
- 单进程访问
- 性能低于原生 Postgres
- 不支持 RLS（行级安全）

## Postgres 引擎

`src/core/postgres-engine.ts` — 基于 postgres.js 的生产级引擎：

- **连接池**：默认 10 个连接，可通过 `GBRAIN_POOL_SIZE` 调整
- **pgvector**：向量索引（IVFFlat / HNSW）
- **RLS**：行级安全，支持团队共享
- **准备语句**：自动检测 PgBouncer 端口（6543），禁用 prepared statements

### Supabase 兼容

自动检测 Supabase PgBouncer 端口并调整行为：
```
端口 6543 → prepare: false（避免 "prepared statement does not exist" 错误）
可通过 GBRAIN_PREPARE=true 环境变量覆盖
```

## Schema 管理

### 嵌入式 Schema

`src/schema-embedded.ts` — 将 `schema.sql` 编译为 TypeScript 常量，避免运行时文件读取。

### PGLite 特有 Schema

`src/core/pglite-schema.ts` — PGLite 不支持的 Postgres 特性的替代方案。

### Schema 验证

`src/core/schema-verify.ts` — 启动时验证数据库 Schema 与代码期望一致。

### 迁移

`src/core/migrate.ts` — 版本化的增量迁移：
```
gbrain apply-migrations --yes
  → 读取当前 schema_version
  → 按顺序执行增量迁移脚本
  → 更新 schema_version
```

## 重试机制

`src/core/retry.ts` — 数据库操作的重试策略：

```typescript
withRetry(operation, BULK_RETRY_OPTS)
  → 指数退避重试
  → 区分可重试错误（连接超时）和不可重试错误（语法错误）
  → 批量操作审计日志
```

## 核心表结构

| 表名 | 说明 |
|------|------|
| `pages` | 页面存储（slug, type, content, frontmatter, compiled_truth） |
| `content_chunks` | 内容块 + 向量嵌入 |
| `page_links` | 页面间链接（wikilinks、markdown links） |
| `entities` | 实体 |
| `facts` | 事实声明（带指标、时间、可见性） |
| `timeline_entries` | 时间线条目 |
| `sources` | 数据源配置（JSONB config） |
| `files` | 二进制文件元数据 |
| `edges` | 知识图谱边（typed edges） |
| `ingest_log` | 摄入日志 |
| `page_versions` | 页面版本历史 |
| `raw_data` | 原始数据存档 |
