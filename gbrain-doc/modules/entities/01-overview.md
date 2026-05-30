# entities — 概述

> 实体系统是 GBrain 知识图谱的基础，自动从内容中识别、解析和管理结构化实体。

## 实体生命周期

```
内容摄入 → NER 提取 → 实体解析 → 页面创建 → 关系建立
```

## 实体消歧

`resolve.ts` — 处理同一实体的不同称呼：
- "Alice" / "Alice Chen" / "A. Chen" → 同一人物
- "Acme" / "Acme AI" / "Acme Inc" → 同一公司
- 基于上下文和已有实体列表进行消歧

## 实体合并

当发现重复实体时：
- 保留最完整的页面
- 合并所有关系边
- 合并时间线
- 更新所有引用

## 与页面的关系

- 每个实体对应一个页面（type: person / company / ...）
- 实体页面包含 Facts、Timeline、Connections
- 实体页面可被搜索、引用、关联
