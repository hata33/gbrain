# embedding — 概述

> 嵌入服务将文本/图片转换为向量，是语义搜索和知识图谱的基础。

## 嵌入模型

| 模型 | 维度 | 用途 |
|------|------|------|
| text-embedding-3-small | 1536 | 默认文本嵌入 |
| text-embedding-3-large | 3072 | 高精度文本嵌入 |
| Gemini embedding | 768 | Google 嵌入 |

## 预检机制

`embed-preflight.ts` — 嵌入前检查：
- API Key 是否可用
- 模型是否可达
- 维度是否一致

## 跳过条件

`embed-skip.ts` — 某些内容不需要嵌入：
- 太短的文本（< 10 字符）
- 纯代码块
- 已有嵌入且内容未变

## 过期检测

`embed-stale.ts` — 检测嵌入是否过期：
- 嵌入模型更换后 → 所有嵌入过期
- 内容更新后 → 对应 chunk 嵌入过期
- 需要重新嵌入（`gbrain reindex`）

## 成本估算

`embedding-pricing.ts` — 估算嵌入操作的 API 成本：
- 按 token 计费
- 支持批量估算
- 在 `gbrain doctor --remediation-plan` 中显示预估成本
