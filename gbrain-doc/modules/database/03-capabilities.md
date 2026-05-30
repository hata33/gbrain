# database — 能力清单

> 两种数据库引擎的能力对比和各自支持的特性。

## 能力对比

| 能力 | PGLite | Postgres |
|------|--------|----------|
| 页面 CRUD | ✓ | ✓ |
| 全文搜索（pg_trgm） | ✓ | ✓ |
| 向量搜索（pgvector） | ✓ | ✓ |
| 图遍历（递归 CTE） | ✓ | ✓ |
| 事务 | ✓ | ✓ |
| 批量操作 | ✓ | ✓ |
| 保留连接 | 透传（无额外开销） | 保留后端连接 |
| 行级安全（RLS） | ✗ | ✓ |
| 多进程并发 | ✗（文件锁保护） | ✓（连接池） |
| 向量索引（IVFFlat/HNSW） | ✗ | ✓ |
| 准备语句 | ✗ | ✓（可选） |
| JSONB 操作 | ✓ | ✓ |
| Schema 迁移 | ✓ | ✓ |
| 文件锁 | ✓（pglite-lock.ts） | N/A |

## 核心 SQL 操作

### 页面操作

```sql
-- Upsert 页面
INSERT INTO pages (source_id, slug, type, content, frontmatter, compiled_truth, ...)
VALUES ($1, $2, $3, $4, $5, $6, ...)
ON CONFLICT (source_id, slug) DO UPDATE SET ...

-- 软删除
UPDATE pages SET deleted_at = now() WHERE slug = $1 AND source_id = $2

-- 批量删除
DELETE FROM pages WHERE slug = ANY($1::text[]) AND source_id = $2
```

### 搜索

```sql
-- 混合搜索：向量 + 全文 + 图谱
WITH vector_results AS (
  SELECT slug, 1 - (embedding <=> $1::vector) AS vec_score
  FROM content_chunks ORDER BY embedding <=> $1::vector LIMIT $2
),
text_results AS (
  SELECT slug, similarity(content, $3) AS text_score
  FROM pages WHERE content % $3 LIMIT $2
)
SELECT ... FROM vector_results FULL JOIN text_results ...
```

### 图遍历

```sql
-- BFS 递归查询
WITH RECURSIVE graph AS (
  SELECT slug, 0 AS depth FROM pages WHERE slug = $1
  UNION
  SELECT pl.target_slug, g.depth + 1
  FROM graph g JOIN page_links pl ON g.slug = pl.source_slug
  WHERE g.depth < $2
) SELECT * FROM graph;
```

## 连接配置

### PGLite

```typescript
{
  engine: 'pglite',
  dataDir: '~/.gbrain/data',  // 数据目录
}
```

### Postgres

```typescript
{
  engine: 'postgres',
  connectionString: 'postgresql://user:pass@host:5432/gbrain',
  poolSize: 10,               // 可选，连接池大小
}
```

## 环境变量

| 变量 | 说明 |
|------|------|
| `GBRAIN_POOL_SIZE` | Postgres 连接池大小（默认 10） |
| `GBRAIN_PREPARE` | 强制启用/禁用准备语句（`true`/`false`） |
| `DATABASE_URL` | Postgres 连接字符串（备用） |
