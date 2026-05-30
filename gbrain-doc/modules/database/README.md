# database — 数据库引擎

> GBrain 的数据库层支持 PGLite（嵌入式）和 Postgres（生产级）两种后端，
> 共享 Schema、SQL 查询和迁移机制。

## 文件结构

| 文件 | 说明 |
|------|------|
| [README.md](README.md) | 本文件，索引与概述 |
| [01-overview.md](01-overview.md) | 双引擎架构、Schema 管理 |
| [02-lifecycle.md](02-lifecycle.md) | 连接、初始化、迁移、断开 |
| [03-capabilities.md](03-capabilities.md) | 能力对比、核心 SQL、配置 |
| [04-policies.md](04-policies.md) | 引擎选择、并发、安全 |

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/db.ts` | Postgres 连接管理 |
| `src/core/pglite-engine.ts` | PGLite 引擎实现 |
| `src/core/postgres-engine.ts` | Postgres 引擎实现 |
| `src/core/pglite-schema.ts` | PGLite 特有 Schema |
| `src/core/pglite-lock.ts` | PGLite 文件锁 |
| `src/core/schema-embedded.ts` | 嵌入式 Schema SQL |
| `src/core/schema-verify.ts` | Schema 验证 |
| `src/core/migrate.ts` | Schema 迁移 |
| `src/core/retry.ts` | 重试机制 |
