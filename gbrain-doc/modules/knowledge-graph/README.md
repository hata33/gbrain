# knowledge-graph — 知识图谱

> GBrain 的知识图谱通过零 LLM 调用的规则提取，自动建立实体间的类型化关系边。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/link-extraction.ts` | 链接/关系提取（纯规则） |

## 概述

```
页面内容: "Alice works at Acme as engineering lead"
  → 规则提取 → 边: { from: "people/alice", to: "companies/acme", type: "works_at" }
  → 零 LLM 成本
  → 写入 edges 表
```

### 边类型

| 类型 | 示例 | 说明 |
|------|------|------|
| `works_at` | Alice → Acme | 工作关系 |
| `invested_in` | Bob → Acme | 投资关系 |
| `founded` | Carol → Acme | 创立关系 |
| `advises` | Dave → Acme | 顾问关系 |
| `attended` | Alice → meeting-2026-03-15 | 参加会议 |
| `mentioned_in` | Alice → note-2026-04-22 | 被提及 |

### 提取机制

1. **Markdown 链接** — `[[Alice]]` 在 Acme 页面 → `works_at` 边
2. **Frontmatter 字段** — `connections: [alice, bob]` → 边
3. **正则匹配** — "X works at Y" / "X invested in Y" → 边
4. **页面类型推断** — person 页面在 companies/ 目录下 → 关联边

### 图遍历

```sql
-- 递归 CTE 实现 BFS 图遍历
WITH RECURSIVE graph AS (
  SELECT slug, 0 AS depth FROM pages WHERE slug = 'people/alice'
  UNION
  SELECT e.target_slug, g.depth + 1
  FROM graph g JOIN edges e ON g.slug = e.source_slug
  WHERE g.depth < 3
) SELECT * FROM graph;
```

### 搜索集成

`graph-signals.ts` — 图谱信号融入搜索排序：
- 关系密度高的实体在搜索中获得加权
- 支持 "who works at Acme?" 这类关系查询
- 纯向量搜索无法回答的问题，图谱可以
