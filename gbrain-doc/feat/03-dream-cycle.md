# 03 — 梦境循环（Dream Cycle）

> GBrain 的梦境循环是 9 阶段自动化维护流程，在用户"睡觉"时执行，
> 醒来时大脑已经完成同步、提取、嵌入和巩固。

## 设计思想

```
手动维护不可持续 — 用户不可能每天手动运行 sync、extract、embed...

解决方案：让大脑在你睡觉时自动工作
  → gbrain dream（一次性）
  → gbrain autopilot（守护进程）
  → Minions 队列化执行
```

## 9 阶段管线

```
Phase 1: lint --fix           → 文件系统修复（Markdown lint）
Phase 2: backlinks --fix      → 反向链接修复（wikilink 解析）
Phase 3: sync                 → 增量同步到数据库
Phase 4: synthesize           → 转录 → 页面（会议录音等）
Phase 5: extract              → 提取实体/事实/链接
Phase 6: patterns             → 跨会话主题分析
Phase 7: emotional_weight     → 情感权重计算
Phase 8: embed --stale        → 过期 chunk 重新嵌入
Phase 9: orphans              → 孤儿页面报告
```

## 阶段依赖

```
lint → sync（先修复文件，再同步到 DB）
sync → extract（先有最新数据，再提取）
extract → embed（先有新 chunk，再嵌入）
extract → patterns（先有最新图谱，再分析模式）
```

## 调用方式

| 方式 | 命令 | 说明 |
|------|------|------|
| CLI 一次性 | `gbrain dream` | 运行一次完整周期 |
| CLI 守护进程 | `gbrain autopilot` | 持续运行 |
| Minions | `autopilot-cycle` | 队列化执行，支持重试 |
| Cron | 外部调度 | 定时触发 |

## 阶段保护

Phase 4 (synthesize)、Phase 6 (patterns)、Phase 7 (consolidate) 被标记为 PROTECTED：
- 只有本地 CLI 调用者可以触发
- MCP 远程调用者不能触发
- 安全考虑：防止远程 Agent 执行高权限操作

## 核心代码

| 文件 | 职责 |
|------|------|
| `src/core/cycle.ts` | 循环核心 |
| `src/core/cycle/` | 循环子模块（23 个文件） |
