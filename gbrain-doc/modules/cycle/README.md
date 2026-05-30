# cycle — 梦境循环

> GBrain 的梦境循环（Dream Cycle）是大脑的自动化维护流程，
> 执行同步、提取、嵌入、综合、巩固等一系列操作，保持大脑健康。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/cycle.ts` | 循环核心（runCycle） |
| `src/core/cycle/` | 循环子模块（23 个文件） |

## 概述

"梦境循环"是 GBrain 最核心的自动化机制——你睡觉时，大脑在工作：

```
gbrain dream
  → 运行完整的维护周期
  → 你醒来时大脑已经更新、提取、巩固了新知识
```

### 阶段顺序

```
Phase 1: lint --fix           → 文件系统修复（无 DB）
Phase 2: backlinks --fix      → 反向链接修复（无 DB）
Phase 3: sync                 → DB 同步变更文件
Phase 4: synthesize           → 转录 → 页面
Phase 5: extract              → 提取实体/事实/链接
Phase 6: patterns             → 跨会话主题分析
Phase 7: emotional_weight     → 情感权重计算
Phase 8: embed --stale        → 过期 chunk 重新嵌入
Phase 9: orphans              → 孤儿页面报告
```

### 设计原则

1. **先修复文件，再索引到 DB** — lint 和 backlinks 在 sync 之前
2. **先提取，再嵌入** — extract 产生的新 chunk 需要嵌入
3. **先同步，再综合** — synthesize 需要最新的页面数据
4. **模式在提取之后** — patterns 依赖最新的图谱状态

### 调用方式

| 方式 | 命令 | 说明 |
|------|------|------|
| CLI 一次性 | `gbrain dream` | 运行一次完整周期 |
| CLI 守护进程 | `gbrain autopilot` | 持续运行，定时执行 |
| Minions | `autopilot-cycle` handler | 队列化执行，支持重试 |
| Cron | 外部 cron 调度 | 定时触发 |

### 阶段保护

Phase 4 (synthesize)、Phase 6 (patterns)、Phase 7 (consolidate) 被标记为 PROTECTED — 只有本地 CLI 调用者可以触发，MCP 远程调用者不能。
