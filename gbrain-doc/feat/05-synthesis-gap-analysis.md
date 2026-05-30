# 05 — 综合与差距分析

> GBrain 的核心差异化能力：不只是搜索页面，而是综合多源信息生成带引用的回答，
> 并明确标注"大脑不知道什么"。

## 问题

传统知识工具返回搜索结果列表：
```
1. people/alice — Alice runs engineering at Acme...
2. meetings/2026-03-15 — Q1 product review...
3. notes/2026-04-22 — Quick pricing chat...
```
用户还需要自己打开每个页面阅读和综合。

## 解决方案

GBrain 返回综合回答：
```
Alice runs engineering at Acme. You last spoke on April 22.
Three things are still open:

1. She owes you the security review (deadline May 1; no update).
2. You committed to pricing for a 500-seat tier (sent April 25; no response).
3. She mentioned hiring a CISO; you said you'd intro someone.

Heads up: nothing's been added since April 22, six weeks ago.
She may have replied through email or Slack DM, channels the brain doesn't see.
```

## 差距分析

每个回答末尾的 "Heads up" 部分是差距分析：
- 告诉用户大脑不知道什么
- 提示可能的信息来源
- 避免基于过时信息做决策

## 与 search 的区别

| 维度 | search | think（综合） |
|------|--------|---------------|
| 输出 | 结果列表 | 综合回答 |
| 引用 | 无 | 每个声明有来源 |
| 差距分析 | 无 | 标注知识空白 |
| LLM 调用 | 可选 | 必须 |

## 核心代码

| 文件 | 职责 |
|------|------|
| `src/core/think/` | 深度推理模块 |
| `src/commands/think.ts` | think CLI 命令 |
