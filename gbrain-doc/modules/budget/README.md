# budget — 预算管理

> GBrain 的预算系统追踪和限制 AI API 调用成本，防止意外超支。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/budget/budget-tracker.ts` | 预算追踪器 |

## 概述

GBrain 的 AI 调用（嵌入、LLM 生成、转录）都有成本，预算系统确保不超支：

```
API 调用
  → budget-tracker 记录成本
  → 累计成本 vs 预算上限
  → 超过预算 → 暂停任务
  → 报告剩余预算
```

### 预算配置

```yaml
budget:
  maxUsd: 5.0             # 每次 dream cycle 最大 $5
  dailyMaxUsd: 20.0       # 每天最大 $20
  alertThreshold: 0.8     # 80% 时告警
```

### 成本估算

- 嵌入：按 token 数 × 模型单价
- LLM 生成：按输入/输出 token 数 × 模型单价
- 转录：按时长 × 单价

### 集成点

- `gbrain doctor --remediation-plan` — 显示预估成本
- `gbrain doctor --remediate --max-usd 5` — 设置成本上限
- Minions — 每个任务追踪成本
- Cycle — 整个周期追踪总成本
