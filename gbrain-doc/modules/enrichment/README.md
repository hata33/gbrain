# enrichment — 内容富化

> GBrain 的富化系统自动为实体页面补充上下文信息（来自外部 API、会议、邮件等），
> 使实体页面从"提到过"升级为"深入了解"。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/enrichment-service.ts` | 富化服务入口 |
| `src/core/enrichment/budget.ts` | 富化预算控制 |
| `src/core/enrichment/completeness.ts` | 完整度评估 |

## 概述

```
实体页面 "people/alice"
  → 富化服务检查完整度
  → 发现缺失信息（工作经历、社交链接、最近互动）
  → 调用外部 API 补充
  → 更新页面的 Facts、Timeline
  → 评估富化后的完整度分数
```

### 富化层级

| 层级 | 来源 | 成本 |
|------|------|------|
| Tier 1 | 大脑内部数据（已有页面、链接） | 零 |
| Tier 2 | 已摄入的外部数据（邮件、日历） | 零 |
| Tier 3 | 外部 API（LinkedIn、Crunchbase） | 有 API 成本 |

### 预算控制

`enrichment/budget.ts` — 控制富化的 API 成本：
- 每次富化的最大 API 调用次数
- 每天的最大富化次数
- 可配置成本上限
