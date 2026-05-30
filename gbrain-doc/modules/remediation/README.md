# remediation — 自我修复

> GBrain 的修复系统自动检测和修复大脑的健康问题，
> 包括 Schema 不一致、孤儿页面、损坏的链接等。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/remediation/index.ts` | 修复入口 |
| `src/core/remediation/plan.ts` | 修复计划 |
| `src/core/remediation/run.ts` | 修复执行 |
| `src/core/remediation/context.ts` | 修复上下文 |
| `src/core/remediation/types.ts` | 类型定义 |

## 概述

```bash
gbrain doctor --remediation-plan --json
  → 预览修复计划
  → 显示将要修复的问题
  → 估算 API 成本

gbrain doctor --remediate --yes --target-score 90 --max-usd 5
  → 执行修复
  → 按依赖顺序执行（sync → extract → embed → consolidate）
  → 每步后重新检查分数
  → 超过成本上限时停止
```

### 修复阶段

```
1. Schema 检查 → 自动迁移
2. 孤儿页面 → 清理或重建链接
3. 损坏的链接 → 重新解析
4. 过期嵌入 → 重新嵌入
5. 缺失的实体 → 重新提取
6. 缺失的综合 → 重新生成
```

### 健康评分

```
doctor 输出健康评分（0-100）：
  → 目标分数（如 90 分）
  → 修复计划显示距离目标还差多少
  → 空大脑或未配置嵌入 → max_reachable_score 上限
```
