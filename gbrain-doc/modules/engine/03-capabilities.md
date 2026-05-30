# engine — 能力清单

> BrainEngine 接口的完整能力清单，按操作域分类。

## 页面操作（Pages CRUD）

| 方法 | 说明 |
|------|------|
| `getPage(slug)` | 按 slug 获取单个页面 |
| `putPage(slug, input)` | 插入或更新页面（upsert） |
| `deletePage(slug)` | 硬删除单个页面（级联删除 chunk/links） |
| `deletePages(slugs, opts)` | 批量硬删除（单条 SQL，原子事务） |
| `softDeletePage(slug)` | 软删除（设置 deleted_at，72h 后自动清除） |
| `restorePage(slug)` | 恢复软删除的页面 |
| `purgeDeletedPages(hours)` | 清除超过指定小时数的软删除页面 |
| `listPages(filters)` | 列出页面（支持类型、源、时间过滤） |
| `resolveSlugs(partial)` | 模糊 slug 解析（支持源范围限制） |
| `getAllSlugs(opts)` | 获取所有 slug 集合 |
| `listAllPageRefs()` | 跨源页面枚举（slug + source_id） |
| `findDuplicatePage(sourceId, opts)` | 基于内容哈希/frontmatter ID 的去重检查 |

## 搜索操作

| 方法 | 说明 |
|------|------|
| `search(query, opts)` | 混合搜索（向量 + 全文 + 图谱） |
| `searchChunks(query, opts)` | chunk 级别搜索 |

## 图谱操作

| 方法 | 说明 |
|------|------|
| `addLinks(links)` | 批量添加页面间链接 |
| `getLinksFrom(slug)` | 获取页面的出链 |
| `getLinksTo(slug)` | 获取页面的入链 |
| `traverseGraph(start, depth)` | BFS 图遍历（支持 frontier cap） |
| `findPath(from, to)` | 两点间最短路径 |

## 时间线操作

| 方法 | 说明 |
|------|------|
| `addTimelineEntry(entry)` | 添加时间线条目 |
| `getTimeline(slug, opts)` | 获取页面的时间线 |
| `findTimelineByDateRange(from, to)` | 按日期范围查询 |

## 事实操作

| 方法 | 说明 |
|------|------|
| `putFact(fact)` | 写入事实声明 |
| `getFacts(slug, opts)` | 获取页面的事实 |
| `findFactsByMetric(metric)` | 按指标类型查找事实 |

## 嵌入操作

| 方法 | 说明 |
|------|------|
| `storeEmbedding(chunkId, vector)` | 存储 chunk 的向量嵌入 |
| `getEmbedding(chunkId)` | 获取 chunk 的向量 |
| `searchByVector(vector, opts)` | 向量相似度搜索 |

## 源管理

| 方法 | 说明 |
|------|------|
| `listAllSources(opts)` | 列出所有数据源 |
| `updateSourceConfig(sourceId, patch)` | 原子更新源配置（JSONB merge） |

## 文件操作

| 方法 | 说明 |
|------|------|
| `storeFile(fileRow)` | 存储二进制文件元数据 |
| `getFilesByPage(slug)` | 获取页面关联的文件 |

## 统计与健康

| 方法 | 说明 |
|------|------|
| `getBrainStats()` | 大脑统计（页面数、chunk 数、源数等） |
| `getBrainHealth()` | 大脑健康状态 |

## 采样操作

| 方法 | 说明 |
|------|------|
| `listPrefixSampledPages(opts)` | 按前缀分层采样（用于 brainstorm/lsd） |
| `listCorpusSample(opts)` | 语料库随机采样 |

## 批次大小常量

`engine-constants.ts` 定义了关键的批次限制：

```typescript
const DELETE_BATCH_SIZE = 500;  // 批量删除每批最多 500 条
const LINK_BATCH_SIZE = 1000;   // 批量添加链接每批最多 1000 条
```
