# engine — 生命周期

> BrainEngine 从创建到销毁的完整生命周期：工厂创建 → 连接 → Schema 初始化 → 事务操作 → 断开。

## 生命周期总览

```
createEngine(config)        ← 工厂创建引擎实例
    │
    ▼
engine.connect(config)      ← 建立数据库连接
    │
    ▼
engine.initSchema()         ← 创建/迁移 Schema
    │
    ▼
engine.transaction(async () ← 业务操作（事务内）
  → putPage / getPage / search / ...
)
    │
    ▼
engine.disconnect()         ← 关闭连接，释放资源
```

## 1. 引擎创建

```typescript
// engine-factory.ts
const engine = await createEngine({ engine: 'pglite' });
// 或
const engine = await createEngine({ engine: 'postgres', connectionString: '...' });
```

工厂通过动态导入避免加载不需要的引擎代码：
- PGLite：导入 `pglite-engine.ts`（含 WASM）
- Postgres：导入 `postgres-engine.ts`（含 postgres.js 驱动）

## 2. 连接

```typescript
await engine.connect({
  engine: 'pglite',
  dataDir: '~/.gbrain/data',
});
```

- **PGLite**：初始化 WASM 运行时，打开本地数据库文件
- **Postgres**：建立连接池，验证 pgvector 扩展可用

## 3. Schema 初始化

```typescript
await engine.initSchema();
```

读取 `src/schema.sql` 并执行建表语句。核心表包括：
- `pages` — 页面存储
- `content_chunks` — 内容块 + 向量
- `page_links` — 页面间链接
- `entities` — 实体
- `facts` — 事实声明
- `timeline_entries` — 时间线条目
- `sources` — 数据源配置
- `files` — 二进制文件元数据
- `edges` — 知识图谱边

Schema 版本通过 `schema_version` 追踪，`gbrain apply-migrations` 执行增量迁移。

## 4. 事务

```typescript
const result = await engine.transaction(async (tx) => {
  await tx.putPage('people/alice', { content: '...' });
  await tx.addLinks([{ from: 'people/alice', to: 'companies/acme' }]);
  return result;
});
```

事务保证原子性：要么全部成功，要么全部回滚。

### 保留连接

```typescript
await engine.withReservedConnection(async (conn) => {
  // Postgres: 使用保留的后端连接
  // PGLite: 直接传递（无额外开销）
});
```

用于长时间运行的操作（如批量导入），避免占用连接池。

## 5. 断开

```typescript
await engine.disconnect();
```

- **PGLite**：关闭 WASM 运行时
- **Postgres**：关闭连接池

## Schema 迁移

GBrain 使用版本化的 Schema 迁移：

```
schema_version: 0 → 需要 apply-migrations
schema_version: 82 → 最新版本（v0.42+）
```

迁移通过 `gbrain apply-migrations --yes` 执行，或由 `gbrain upgrade` 自动触发。
