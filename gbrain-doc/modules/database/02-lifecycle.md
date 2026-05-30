# database — 生命周期

> 数据库引擎从创建到销毁的完整流程，包括连接管理、Schema 初始化和迁移。

## 连接流程

### PGLite

```
createEngine({ engine: 'pglite' })
  → 动态导入 PGLiteEngine
  → engine.connect({ dataDir: '~/.gbrain/data' })
    → 初始化 PGlite WASM 运行时
    → 加载 pgvector 和 pg_trgm 扩展
    → 获取文件锁（pglite-lock.ts）
    → 打开或创建数据库文件
  → engine.initSchema()
    → 执行 PGLite 特有 Schema（pglite-schema.ts）
    → 运行增量迁移
```

### Postgres

```
createEngine({ engine: 'postgres', connectionString: '...' })
  → 动态导入 PostgresEngine
  → engine.connect({ connectionString })
    → 检测端口（6543 = PgBouncer → prepare: false）
    → 创建连接池（默认 10 连接）
    → 验证 pgvector 扩展可用
    → 验证 Schema 版本
  → engine.initSchema()
    → 执行 Schema SQL
    → 运行增量迁移
    → 应用向量索引策略（applyChunkEmbeddingIndexPolicy）
```

## Schema 初始化

```typescript
// src/schema-embedded.ts — Schema 编译为常量
export const SCHEMA_SQL = `CREATE TABLE IF NOT EXISTS pages (...)`;

// 启动时验证
verifySchema(sql)
  → 检查所有必需的表是否存在
  → 检查 schema_version 是否匹配
  → 不匹配 → 报错并提示运行 apply-migrations
```

## 迁移执行

```typescript
// src/core/migrate.ts
runMigrations(engine, config)
  → 读取当前 schema_version
  → 加载迁移脚本列表
  → 按版本号顺序执行
  → 每个迁移在一个事务内完成
  → 更新 schema_version
```

## 连接池管理（Postgres）

```
连接池配置：
  → 默认大小：10（GBRAIN_POOL_SIZE 可覆盖）
  → PgBouncer 端口自动检测
  → 准备语句策略：auto/true/false

连接健康：
  → 自动重连
  → 连接超时检测
  → 空闲连接回收
```

## 断开连接

```typescript
await engine.disconnect();
// PGLite: 释放文件锁，关闭 WASM
// Postgres: 关闭连接池
```

## 错误处理

`src/core/errors.ts` — 统一错误类型：

```typescript
class GBrainError extends Error {
  code: string;       // 错误代码
  cause?: Error;      // 原始错误
}
```

数据库错误通过 `withRetry()` 自动重试，区分可重试和不可重试错误。
