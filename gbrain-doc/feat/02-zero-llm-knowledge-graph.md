# 02 — 零 LLM 知识图谱

> GBrain 通过规则提取自动建立实体间的关系边，零 LLM 调用，
> 使得知识图谱构建可以大规模运行，不产生 API 成本。

## 问题

传统知识图谱构建需要 LLM 提取实体关系：
- API 成本高（每个页面都需要 LLM 调用）
- 延迟大（LLM 推理需要时间）
- 不确定性（LLM 可能提取错误）

## 解决方案

GBrain 使用纯规则提取知识图谱边：

```
页面内容: "Alice works at Acme as engineering lead"
  → 正则匹配 "X works at Y" 模式
  → 提取: { from: "people/alice", to: "companies/acme", type: "works_at" }
  → 零 LLM 成本
  → 毫秒级延迟
```

## 边类型

| 类型 | 提取方式 | 示例 |
|------|----------|------|
| `works_at` | 正则 + frontmatter | Alice → Acme |
| `invested_in` | 正则 + frontmatter | Bob → Acme |
| `founded` | 正则 + frontmatter | Carol → Acme |
| `advises` | 正则 + frontmatter | Dave → Acme |
| `attended` | 会议页面参与者列表 | Alice → meeting-2026-03 |
| `mentioned_in` | Markdown 链接 | Alice → note-2026-04 |

## 提取机制

### 1. Markdown 链接

```markdown
在 [[Acme]] 的会议上见到了 [[Alice]]
  → 提取: Alice mentioned_in note-xxx
  → 提取: Acme mentioned_in note-xxx
```

### 2. Frontmatter 字段

```yaml
---
connections:
  - name: Alice
    role: engineering lead
    company: Acme
---
```

### 3. 正则模式

```
"Alice works at Acme"     → works_at
"Bob invested in Acme"    → invested_in
"Carol founded Acme"      → founded
"Dave advises Acme"       → advises
```

### 4. 页面类型推断

```
people/alice 页面
  → 在 companies/acme 的 connections 中出现
  → 自动建立 works_at 边
```

## 与搜索的集成

图谱信号融入搜索排序（`graph-signals.ts`）：
- 关系密度高的实体在搜索中获得加权
- 支持 "who works at Acme?" 这类关系查询
- 纯向量搜索无法回答的问题，图谱可以

## Benchmark

在 240 页语料库上：
- 图谱信号带来 **+31.4 个百分点 P@5** 提升
- 零额外 API 成本

## 核心代码

| 文件 | 职责 |
|------|------|
| `src/core/link-extraction.ts` | 链接/关系提取 |
| `src/core/search/graph-signals.ts` | 图谱信号融入搜索 |
