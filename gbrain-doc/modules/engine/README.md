# engine — 核心脑引擎

> BrainEngine 是 GBrain 的核心抽象接口，定义了知识大脑的所有数据操作，
> 支持 PGLite（嵌入式）和 Postgres（生产级）两种后端引擎。

## 文件结构

| 文件 | 说明 |
|------|------|
| [README.md](README.md) | 本文件，索引与概述 |
| [01-overview.md](01-overview.md) | 设计思想、架构定位 |
| [02-lifecycle.md](02-lifecycle.md) | 引擎创建、连接、Schema 初始化、事务 |
| [03-capabilities.md](03-capabilities.md) | BrainEngine 接口完整能力清单 |
| [04-policies.md](04-policies.md) | 引擎选择策略、多源隔离、软删除 |

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/engine.ts` | BrainEngine 接口定义（1927 行） |
| `src/core/engine-factory.ts` | 引擎工厂，按配置创建实例 |
| `src/core/pglite-engine.ts` | PGLite 引擎实现 |
| `src/core/postgres-engine.ts` | Postgres 引擎实现 |
| `src/core/engine-constants.ts` | 引擎常量（批次大小等） |
| `src/core/types.ts` | 共享类型定义 |
