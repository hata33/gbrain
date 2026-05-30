# calibration — 质量校准

> GBrain 的校准系统评估大脑的预测准确性和一致性，
> 生成校准曲线和置信度报告。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/calibration/cross-brain.ts` | 跨大脑校准 |
| `src/core/calibration/take-forecast.ts` | 预测追踪 |
| `src/core/calibration/domain-aggregators.ts` | 领域聚合 |
| `src/core/calibration/nudge.ts` | 校准提示 |
| `src/core/calibration/recall-footer.ts` | recall 页脚附加 |

## 概述

校准系统回答一个核心问题：**大脑的预测有多准确？**

```
gbrain eval calibration
  → 收集所有 takes（观点/赌注）
  → 检查哪些已被验证
  → 计算校准曲线
  → 输出置信度报告
```

### Founder Scorecard

```bash
gbrain founder scorecard <entity>
  → 四维评分：
    - claim_accuracy: 声明准确率
    - consistency: 一致性（前后是否矛盾）
    - growth_trajectory: 增长轨迹
    - red_flags: 红旗信号
```

### 校准曲线

显示不同置信度级别的预测实际准确率：
- 置信度 80% 的预测 → 实际准确率应接近 80%
- 如果偏离 → 大脑需要校准

### 跨大脑校准

`cross-brain.ts` — 比较多个大脑的校准状态，用于团队协作场景。
