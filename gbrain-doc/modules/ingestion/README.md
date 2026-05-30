# ingestion — 内容摄入管线

> GBrain 的摄入系统负责从各种来源（本地文件、邮件、日历、网页等）
> 将内容导入大脑，包括去重、切块、嵌入和元数据提取。

## 文件结构

| 文件 | 说明 |
|------|------|
| [README.md](README.md) | 本文件 |
| [01-overview.md](01-overview.md) | 摄入架构与源类型 |
| [02-lifecycle.md](02-lifecycle.md) | 完整摄入流程 |
| [03-capabilities.md](03-capabilities.md) | 能力清单 |
| [04-policies.md](04-policies.md) | 去重策略与错误处理 |

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/ingestion/index.ts` | 公共 API 导出 |
| `src/core/ingestion/types.ts` | 类型定义 |
| `src/core/ingestion/daemon.ts` | 摄入守护进程 |
| `src/core/ingestion/dedup.ts` | 去重逻辑 |
| `src/core/ingestion/sources/` | 内置源实现 |
| `src/core/import-file.ts` | 文件导入 |
| `src/core/sync.ts` | 同步引擎 |
