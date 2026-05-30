# brain-registry — 多脑管理

> GBrain 的大脑注册表管理多个独立的知识大脑（brain），
> 支持按 brain 路由操作，实现个人脑 + 团队脑的联邦架构。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/brain-registry.ts` | 大脑注册表 |
| `src/core/brain-resolver.ts` | CLI 命令的大脑解析 |

## 概述

```
一个用户可以有多个大脑：
  → 'host'：主大脑（~/.gbrain/config.json 定义）
  → 'work'：工作大脑（mount）
  → 'team'：团队共享大脑（mount）

每个大脑是独立的数据库，有独立的页面、实体、事实。
```

### 注册表架构

```
BrainRegistry
  ├── 'host' → ~/.gbrain/data (PGLite)
  ├── 'work' → ~/work-brain (PGLite)
  └── 'team' → postgres://team-db (Postgres)
```

### 大脑解析优先级

```
1. --brain <id> 参数
2. GBRAIN_BRAIN_ID 环境变量
3. .gbrain-mount dotfile（CWD 或祖先目录）
4. mounts.json 中 path 匹配 CWD 的条目
5. 默认 brain（config.json brains.default）
6. 'host' 回退（兼容性）
```

### 挂载配置

`~/.gbrain/mounts.json`：
```json
{
  "work": {
    "path": "~/work-brain",
    "engine": "pglite"
  },
  "team": {
    "db_url": "postgres://...",
    "engine": "postgres"
  }
}
```

### 安全隔离

- 每个大脑独立的 RLS 策略（Postgres）
- 跨大脑查询只在 latent-space 层面（搜索结果融合）
- 不同大脑的数据物理隔离
