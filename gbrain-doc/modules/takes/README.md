# takes — 观点提取

> GBrain 的 Takes 系统提取和追踪主观观点、判断和赌注，
> 用于校准预测准确性和一致性。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/takes-fence.ts` | Takes 围栏解析/渲染 |
| `src/core/takes-resolution.ts` | 观点解析与合并 |
| `src/core/takes-quality-eval/` | 观点质量评估 |

## 概述

Takes 是 GBrain 中的主观判断记录，与 Facts（客观事实）互补：

```markdown
## Takes
<!--- gbrain:takes:begin -->
| # | claim | kind | who | weight | since | source |
|---|-------|------|-----|--------|-------|--------|
| 1 | CEO of Acme | fact | world | 1.0 | 2017-01 | Crustdata |
| 2 | Strong technical founder | take | garry | 0.85 | 2026-04-29 | meeting |
| 3 | Will reach $50B | bet | garry | 0.7 | 2026-04-29 → 2026-06 | superseded by #4 |
<!--- gbrain:takes:end -->
```

### Takes 类型

| 类型 | 说明 | 示例 |
|------|------|------|
| `fact` | 客观事实 | "CEO of Acme" |
| `take` | 主观判断 | "Strong technical founder" |
| `bet` | 预测/赌注 | "Will reach $50B" |

### 围栏格式

Markdown 是 source of truth（git 是规范），DB 的 takes 表是派生索引。围栏使用 HTML 注释标记边界。

### 预测追踪

- 赌注可以被 superseded（取代）
- 支持时间范围（since → until）
- 校准系统追踪预测准确率
