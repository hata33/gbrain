# 04 — 结构化知识（Facts + Takes）

> GBrain 的双轨知识体系：Facts（客观事实）和 Takes（主观判断），
> 让大脑同时理解"是什么"和"怎么看"。

## 设计思想

传统知识系统只存储客观事实，但人类决策还需要主观判断：
- "Acme 的 MRR 是 $50K" → Fact（客观）
- "Acme 是一个 strong technical founder" → Take（主观）
- "Acme 会在 Q3 达到 $100K MRR" → Take/Bet（预测）

GBrain 同时追踪两者，支持校准和趋势分析。

## Facts 系统

### 结构化格式

```markdown
## Facts
| metric | value | unit | period | source |
|--------|-------|------|--------|--------|
| mrr | 50000 | USD | monthly | 2026-03-15 meeting |
| employees | 25 | count | - | 2026-04-01 email |
```

### 特性
- 从非结构化内容自动提取
- 支持时间线追踪（指标变化）
- 支持衰减（旧事实降低可信度）
- 支持遗忘（过时事实被清理）

## Takes 系统

### 结构化格式

```markdown
## Takes
| # | claim | kind | who | weight | since | source |
|---|-------|------|-----|--------|-------|--------|
| 1 | CEO of Acme | fact | world | 1.0 | 2017-01 | Crustdata |
| 2 | Strong technical founder | take | garry | 0.85 | 2026-04-29 | meeting |
| 3 | Will reach $50B | bet | garry | 0.7 | 2026-04-29 → 2026-06 | superseded by #4 |
```

### Takes 类型
- **fact** — 客观事实（与 Facts 系统重叠）
- **take** — 主观判断
- **bet** — 预测/赌注（可追踪准确性）

### 校准系统
- 追踪预测的实际结果
- 计算校准曲线
- Founder Scorecard（四维评分）

## 核心代码

| 文件 | 职责 |
|------|------|
| `src/core/facts/` | Facts 系统（13 个文件） |
| `src/core/takes-fence.ts` | Takes 围栏解析 |
| `src/core/calibration/` | 校准系统 |
