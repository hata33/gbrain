# extract — 内容提取

> GBrain 的提取模块从原始内容中提取实体、事实、时间线和关系，
> 是摄入管线和梦境循环的核心处理步骤。

## 文件结构

| 文件 | 说明 |
|------|------|
| [README.md](README.md) | 本文件 |
| [01-overview.md](01-overview.md) | 提取架构与流程 |

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/extract/receipt-writer.ts` | 提取收据写入 |
| `src/core/extract/rollup-writer.ts` | 提取汇总写入 |
| `src/core/extract-ner.ts` | 命名实体识别 |
| `src/core/extract-takes-from-pages.ts` | 从页面提取观点 |
| `src/core/extract-timeline-from-meetings.ts` | 从会议提取时间线 |

## 概述

提取是 GBrain 将非结构化内容转化为结构化知识的关键步骤：

```
原始内容（Markdown、邮件、会议记录...）
  │
  ├── NER 提取（extract-ner.ts）
  │   → 识别人名、公司名、地名
  │   → 规则 + 正则匹配（零 LLM 成本）
  │
  ├── 事实提取（facts/extract.ts）
  │   → LLM 提取事实声明
  │   → 结构化为 { metric, value, unit, period }
  │
  ├── 时间线提取（extract-timeline-from-meetings.ts）
  │   → 从会议记录提取事件和日期
  │   → 建立时间线条目
  │
  ├── 观点提取（extract-takes-from-pages.ts）
  │   → 提取主观观点和判断
  │   → 用于校准和质量评估
  │
  └── 提取收据（receipt-writer.ts）
      → 记录提取操作的结果
      → 作为 extract_receipt 页面存储
```

## 提取收据

每次提取操作生成一个收据页面（type: `extract_receipt`）：
- 记录提取了多少实体、事实、关系
- 记录提取耗时和成本
- slug 前缀 `extracts/`
- 在搜索中降权（factor 0.3）
