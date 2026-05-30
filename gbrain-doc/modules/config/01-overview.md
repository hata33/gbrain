# config — 概述

> 配置系统管理 GBrain 的所有运行时设置，支持 YAML 文件、环境变量和 CLI 参数三层覆盖。

## 配置分类

| 分类 | 配置项 | 说明 |
|------|--------|------|
| 引擎 | `engine` | pglite / postgres |
| 数据库 | `database.connectionString` | 连接字符串 |
| AI | `ai.provider`, `ai.apiKey` | AI Provider 配置 |
| 搜索 | `search.mode`, `search.reranker` | 搜索行为 |
| 数据源 | `sources[]` | 数据源列表 |
| 梦境 | `dream.enabled`, `dream.schedule` | 自动化循环 |
| 存储 | `storage.tier` | 存储分层策略 |
| 预算 | `budget.maxUsd` | API 成本上限 |

## 环境变量

| 变量 | 说明 |
|------|------|
| `GBRAIN_ENGINE` | 覆盖引擎类型 |
| `DATABASE_URL` | 数据库连接字符串 |
| `OPENAI_API_KEY` | OpenAI API Key |
| `ANTHROPIC_API_KEY` | Anthropic API Key |
| `GBRAIN_EMBEDDING_MODEL` | 覆盖嵌入模型 |
| `GBRAIN_EMBEDDING_DIMENSIONS` | 覆盖嵌入维度 |
| `GBRAIN_POOL_SIZE` | Postgres 连接池大小 |
| `GBRAIN_CONTRIBUTOR_MODE` | 启用贡献者模式（eval 采集） |

## 模型配置

`model-config.ts` — 管理 AI 模型的选择和 fallback：

```yaml
ai:
  provider: "openai"
  model: "gpt-4o"
  fallback:
    - provider: "anthropic"
      model: "claude-3-haiku"
```
