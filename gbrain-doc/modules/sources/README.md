# sources — 数据源管理

> GBrain 的数据源系统管理大脑中的多个数据来源（本地文件夹、远程仓库等），
> 支持独立同步、配置和健康监控。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/sources-load.ts` | 源表加载器（共享查询） |
| `src/core/sources-ops.ts` | 源操作（列表、创建、归档） |
| `src/core/source-config-redact.ts` | 配置脱敏 |
| `src/core/source-health.ts` | 源健康检查 |
| `src/core/source-id.ts` | 源 ID 规范化 |
| `src/core/source-resolver.ts` | CLI 源解析 |

## 概述

一个大脑可以包含多个数据源，每个源是独立的文件集合：

```
brain/
  ├── source: "default"    → ~/my-vault (Obsidian)
  ├── source: "work"       → ~/work-notes
  ├── source: "code"       → ~/projects (代码仓库)
  └── source: "archive"    → ~/old-notes (归档)
```

### 源解析优先级

```
1. --source <id> 参数
2. GBRAIN_SOURCE_ID 环境变量
3. .gbrain-source dotfile
4. 源表中 path 匹配 CWD 的条目
5. 默认源（"default"）
```

### 源健康检查

`source-health.ts` — 检查每个源的健康状态：
- 本地路径是否存在
- 最后同步时间
- 文件数量
- 错误率

### 源配置脱敏

`source-config-redact.ts` — 在日志中脱敏源配置中的敏感信息（API Key、密码等）。
