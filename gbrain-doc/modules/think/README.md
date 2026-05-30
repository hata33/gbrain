# think — 深度推理

> GBrain 的 think 命令执行深度推理，综合大脑中的多源信息生成结构化回答，
> 支持轨迹分析和知识更新意图检测。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/think/` | 深度推理模块（7 个文件） |
| `src/commands/think.ts` | think CLI 命令 |

## 概述

`gbrain think` 是大脑的"深度思考"接口——不是简单的搜索，而是综合分析：

```bash
gbrain think "Alice 上季度的表现如何？"
  → 搜索 Alice 相关页面
  → 检索事实和时间线
  → 综合分析
  → 生成带引用的结构化回答
  → 标注大脑不知道的部分（gap analysis）
```

### 与 search 的区别

| 维度 | search | think |
|------|--------|-------|
| 输出 | 搜索结果列表 | 综合回答 |
| 引用 | 无 | 每个声明有来源 |
| Gap 分析 | 无 | 标注知识空白 |
| LLM 调用 | 可选 | 必须 |
| 耗时 | 快（毫秒级） | 慢（秒级） |

### 意图检测

think 模块自动检测查询意图：
- **temporal** → 时间相关查询，自动使用轨迹数据
- **knowledge_update** → 知识更新查询
- **general** → 通用综合查询

### 轨迹集成

v0.40.2.0+ 的 think 自动使用轨迹数据：
- 当检测到 temporal 意图时
- 自动查询 `find_trajectory` 获取时间线
- 在回答中包含趋势分析
- 可通过 `think.trajectory_enabled=false` 禁用
