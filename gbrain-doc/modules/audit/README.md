# audit — 审计系统

> GBrain 的审计系统记录关键操作的执行日志，用于调试、合规和问题追踪。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/audit/audit-writer.ts` | 审计日志写入 |
| `src/core/audit/batch-retry-audit.ts` | 批量重试审计 |
| `src/core/audit/content-sanity-audit.ts` | 内容完整性审计 |
| `src/core/audit/db-disconnect-audit.ts` | 数据库断开审计 |
| `src/core/audit/lock-renewal-audit.ts` | 锁续期审计 |
| `src/core/audit/redact-connection-info.ts` | 连接信息脱敏 |

## 概述

GBrain 记录关键操作的审计日志：

| 审计类型 | 说明 |
|----------|------|
| 批量重试 | 批量数据库操作的重试记录 |
| 内容完整性 | 内容截断、编码异常 |
| 数据库断开 | 连接意外断开 |
| 锁续期 | 文件锁的获取和续期 |
| 连接脱敏 | 确保日志不泄露连接字符串 |
