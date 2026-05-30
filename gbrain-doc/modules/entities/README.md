# entities — 实体识别与管理

> GBrain 的实体系统自动从内容中识别人物、公司、项目等实体，
> 建立结构化的实体页面和关系网络。

## 文件结构

| 文件 | 说明 |
|------|------|
| [README.md](README.md) | 本文件 |
| [01-overview.md](01-overview.md) | 实体识别架构 |

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/entities/resolve.ts` | 实体解析 |
| `src/core/extract-ner.ts` | NER（命名实体识别） |

## 概述

GBrain 的实体系统是知识图谱的基础。它自动从内容中识别出人名、公司名、项目名等实体，并为每个实体创建独立的页面。

### 实体类型

| 类型 | 示例 slug | 说明 |
|------|-----------|------|
| person | `people/alice` | 人物 |
| company | `companies/acme` | 公司 |
| deal | `deals/series-a-acme` | 交易 |
| project | `projects/gbrain` | 项目 |
| concept | `concepts/rag` | 概念 |

### 实体识别流程

```
内容摄入
  → NER 提取（extract-ner.ts）
  → 规则匹配（正则表达式、模式匹配）
  → LLM 辅助提取（可选）
  → 实体解析（resolve.ts）
    → 消歧（同一实体的不同称呼）
    → 合并（重复实体合并）
  → 创建/更新实体页面
  → 建立关系边
```

### 实体页面

每个实体对应一个标准化页面：
```markdown
---
type: person
name: Alice
connections: 15
---

## Facts
- Works at Acme (engineering lead)
- Last spoke: 2026-04-22

## Timeline
- 2026-04-22: Quick pricing chat
- 2026-03-15: Q1 product review
```

### 零 LLM 提取

GBrain 的知识图谱边（works_at, invested_in, founded 等）通过规则提取，不调用 LLM：
- 正则匹配 "X works at Y" 模式
- Frontmatter 字段映射
- Markdown 链接关系

这使得边提取是零成本的，可以大规模运行。
