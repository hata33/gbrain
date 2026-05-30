# ingestion — 概述

> 摄入系统将外部内容（文件、邮件、日历、网页等）转化为大脑中的标准化页面。

## 设计思想

GBrain 的大脑需要持续"进食"——从各种来源摄入内容：

```
外部来源                摄入管线                 大脑存储
┌──────────┐    ┌──────────────────┐    ┌──────────┐
│ Obsidian │───→│                  │───→│  pages   │
│ vault    │    │  Ingestion       │    │  chunks  │
├──────────┤    │  Pipeline        │    │  edges   │
│ Email    │───→│                  │───→│  facts   │
├──────────┤    │  ┌────────────┐  │    │  entities│
│ Calendar │───→│  │ 去重检查    │  │    └──────────┘
├──────────┤    │  │ 内容切块    │  │
│ Twitter  │───→│  │ 元数据提取  │  │
├──────────┤    │  │ 向量嵌入    │  │
│ 网页     │───→│  │ 实体提取    │  │
└──────────┘    │  └────────────┘  │
                └──────────────────┘
```

## 源类型

### 内置源
- **本地文件夹** — Obsidian vault、Markdown 文件、代码仓库
- **文件导入** — 单个文件导入（import-file.ts）
- **同步引擎** — 增量同步本地文件夹（sync.ts）

### 插件源（IngestionSource 接口）
- **邮件** — Gmail、Outlook
- **日历** — Google Calendar、Outlook Calendar
- **聊天** — Slack、Discord
- **社交** — Twitter/X
- **网页** — RSS、网页抓取
- **自定义** — 用户可编写自定义源

## 公共 API

```typescript
// 供技能包发布者使用
import {
  IngestionSource,      // 源接口
  IngestionEvent,       // 摄入事件
  computeContentHash,   // 内容哈希
} from 'gbrain/ingestion';
```

## 去重

`dedup.ts` — 避免重复导入相同内容：
- 内容哈希比较
- Frontmatter ID 匹配
- Slug 匹配
