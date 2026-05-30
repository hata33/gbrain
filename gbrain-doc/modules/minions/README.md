# minions — 后台工人

> GBrain 的 Minions 系统是持久化的任务队列，支持重试、预算控制、
> 静默时间和背压管理，是 autopilot 和 dream cycle 的执行引擎。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/minions/index.ts` | Minions 入口 |
| `src/core/minions/queue.ts` | 任务队列 |
| `src/core/minions/handlers/` | 任务处理器 |
| `src/core/minions/budget-meter.ts` | 预算计量 |
| `src/core/minions/budget-tracker.ts` | 预算追踪 |
| `src/core/minions/quiet-hours.ts` | 静默时间 |
| `src/core/minions/rate-leases.ts` | 速率控制 |
| `src/core/minions/backoff.ts` | 退避策略 |
| `src/core/minions/error-classify.ts` | 错误分类 |
| `src/core/minions/exit-classification.ts` | 退出分类 |

## 概述

```
gbrain autopilot
  → Minions 守护进程启动
  → 从队列获取任务
  → 检查预算、静默时间、速率
  → 执行任务（sync、extract、embed...）
  → 记录结果
  → 失败 → 重试（指数退避）
  → 循环
```

### 任务类型

| 任务 | 说明 |
|------|------|
| `autopilot-cycle` | 完整的维护周期 |
| `sync` | 文件同步 |
| `extract` | 内容提取 |
| `embed` | 向量嵌入 |
| `enrich` | 实体富化 |
| `transcribe` | 语音转录 |

### 预算控制

```
budget-meter.ts → 每次 API 调用计量
budget-tracker.ts → 追踪总成本
  → 超过预算 → 暂停任务
  → 可配置每小时/每天成本上限
```

### 静默时间

```
quiet-hours.ts → 定义静默时段（如 23:00-07:00）
  → 静默时段内不执行任务
  → 用于减少夜间 API 调用和噪音
```

### 背压控制

```
lease-cap-controller.ts → 控制并发任务数
  → 太多任务排队 → 降低速率
  → 队列清空 → 恢复速率
```
