# extract — 概述

> 提取系统是 GBrain 知识结构化的核心，将非结构化内容转化为实体、事实、时间线和关系。

## 提取层次

```
Level 0: 规则提取（零 LLM 成本）
  → NER（人名、公司名）
  → Frontmatter 解析
  → Markdown 链接提取

Level 1: LLM 辅助提取
  → 事实声明提取
  → 观点提取
  → 时间线提取

Level 2: 深度提取
  → 综合分析（synthesize）
  → 模式识别（patterns）
  → 知识巩固（consolidate）
```

## 提取收据系统

每个提取操作记录为 `extract_receipt` 页面：
- 提取了多少实体/事实/关系
- 耗时和 API 成本
- 成功/失败状态
- 用于审计和质量追踪

## 与 Dream Cycle 的集成

提取是梦境循环（dream cycle）的核心步骤：
```
Dream Cycle:
  → sync（同步文件）
  → extract（提取实体/事实）← 这里
  → embed（嵌入向量）
  → synthesize（综合生成）
  → consolidate（巩固记忆）
```
