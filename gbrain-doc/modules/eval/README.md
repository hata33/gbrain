# eval — 评估框架

> GBrain 的评估框架用于测量搜索质量、提取准确性和系统性能，
> 支持本地评估和公共基准测试。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/eval/` | 评估核心模块 |
| `src/core/eval/drift-watch.ts` | 质量漂移监控 |
| `src/core/eval/metric-glossary.ts` | 指标术语表 |
| `src/eval/` | 公共评估数据集 |
| `src/commands/eval*.ts` | 评估 CLI 命令（15+ 个） |

## 概述

GBrain 有完整的评估体系：

### 评估类型

| 评估 | 命令 | 说明 |
|------|------|------|
| 搜索质量 | `gbrain eval` | 测量搜索精度和召回率 |
| 对话解析 | `gbrain eval conversation-parser` | 对话切分准确性 |
| 事实提取 | `gbrain eval extract-atoms` | 事实提取准确性 |
| 观点质量 | `gbrain eval takes-quality` | 观点提取质量 |
| 跨模态 | `gbrain eval cross-modal` | 文本/图片跨模态搜索 |
| 矛盾检测 | `gbrain eval contradictions` | 矛盾识别准确性 |
| 轨迹 | `gbrain eval trajectory` | 轨迹分析准确性 |
| LongMemEval | `gbrain eval longmemeval` | 公共长记忆基准 |

### 评估流程

```
GBRAIN_CONTRIBUTOR_MODE=1      → 启用评估采集
gbrain eval export --since 7d  → 导出最近 7 天的查询
gbrain eval replay --against base.ndjson  → 重放评估
  → 对比搜索质量变化
  → 输出 P@5、R@5 等指标
```

### 漂移监控

`drift-watch.ts` — 持续监控搜索质量：
- 跟踪 P@5、R@5 等指标的趋势
- 质量下降时告警
- 用于 CI/CD 中的质量门禁
