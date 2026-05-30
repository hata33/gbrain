# config — 配置系统

> GBrain 的配置系统管理大脑的所有运行时设置，包括 AI Provider、搜索模式、
> 存储引擎、数据源和功能开关。

## 文件结构

| 文件 | 说明 |
|------|------|
| [README.md](README.md) | 本文件 |
| [01-overview.md](01-overview.md) | 配置架构与格式 |

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/config.ts` | 配置加载与管理 |
| `src/core/model-config.ts` | 模型配置 |
| `src/core/source-config-redact.ts` | 配置脱敏（日志安全） |
| `gbrain.yml` | 项目根配置文件示例 |

## 概述

GBrain 使用 YAML 配置文件（`gbrain.yml`），支持环境变量替换和多层覆盖。

### 配置结构

```yaml
# gbrain.yml
engine: "pglite"              # 或 "postgres"
database:
  connectionString: "${DATABASE_URL}"

ai:
  provider: "openai"
  apiKey: "${OPENAI_API_KEY}"
  embeddingModel: "text-embedding-3-small"
  embeddingDimensions: 1536

search:
  mode: "hybrid"
  reranker: true
  expansion: true

sources:
  - id: "default"
    path: "~/my-vault"
  - id: "work"
    path: "~/work-notes"

dream:
  enabled: true
  schedule: "0 3 * * *"       # 每天凌晨 3 点
```

### 配置层次

1. **默认值** — 代码中的硬编码默认值
2. **gbrain.yml** — 项目配置文件
3. **环境变量** — `GBRAIN_*` 前缀的环境变量
4. **CLI 参数** — 命令行参数覆盖

### 配置加载

```
loadConfigWithEngine(engine)
  → 读取 gbrain.yml
  → 环境变量替换（${VAR} 语法）
  → 合并默认值
  → 验证配置合法性
  → 返回不可变配置对象
```

### 配置脱敏

`source-config-redact.ts` — 在日志中脱敏敏感配置：
- API Key → `***`
- 连接字符串 → 脱敏密码部分
- 确保日志不泄露凭证
