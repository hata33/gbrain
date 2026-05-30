# 07 — 自我修复（Doctor 系统）

> GBrain 的 Doctor 系统自动检测和修复大脑的健康问题，
> 支持健康评分、修复计划和成本控制的完整自愈流程。

## 设计思想

```
大脑会"生病"：
  → Schema 不一致
  → 孤儿页面（没有链接指向的页面）
  → 过期嵌入（内容更新了但嵌入没更新）
  → 损坏的链接（目标页面不存在）
  → 缺失的综合（页面没有 compiled_truth）

Doctor 系统：
  → 诊断问题
  → 生成修复计划
  → 按依赖顺序执行修复
  → 每步后重新检查分数
  → 超过成本上限时停止
```

## 使用方式

```bash
# 预览修复计划
gbrain doctor --remediation-plan --json

# 执行修复（目标 90 分，最多 $5）
gbrain doctor --remediate --yes --target-score 90 --max-usd 5

# 快速修复（自动检测和修复）
gbrain doctor --fix
```

## 健康评分

```
doctor 输出健康评分（0-100）：
  → 100 分：完美状态
  → 90+ 分：良好
  → 70-90 分：需要关注
  → < 70 分：需要修复
  → 空大脑或未配置嵌入 → max_reachable_score 上限
```

## 修复阶段

```
1. Schema 检查 → 自动迁移
2. 孤儿页面 → 清理或重建链接
3. 损坏的链接 → 重新解析
4. 过期嵌入 → 重新嵌入（API 成本）
5. 缺失的实体 → 重新提取（API 成本）
6. 缺失的综合 → 重新生成（API 成本）
```

## 成本控制

```
修复可能产生 API 成本（嵌入、LLM 调用）
  → --max-usd 参数限制总成本
  → 每步执行前检查剩余预算
  → 超过预算时停止并报告已完成的步骤
```

## 核心代码

| 文件 | 职责 |
|------|------|
| `src/core/remediation/plan.ts` | 修复计划生成 |
| `src/core/remediation/run.ts` | 修复执行 |
| `src/commands/doctor.ts` | Doctor CLI 命令 |
