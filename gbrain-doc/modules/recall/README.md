# recall — 记忆召回与综合

> GBrain 的 recall 命令从大脑中检索事实、实体和对话，
> 以结构化格式返回，可直接注入 LLM 上下文。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/commands/recall.ts` | recall CLI 命令 |
| `src/core/context-engine.ts` | OpenClaw 上下文引擎集成 |

## 概述

`gbrain recall` 是大脑的"记忆提取"接口：

```bash
gbrain recall alice                  # 查询 Alice 的所有事实
gbrain recall --since "8 hours ago"  # 最近 8 小时的事实
gbrain recall --session <id>         # 按会话查询
gbrain recall --today                # 今日事实
gbrain recall --grep "pricing"       # 文本过滤
gbrain recall --as-context           # 输出为 LLM 可用的 Markdown
```

### 与 MCP recall 的关系

- CLI `recall` 和 MCP `recall` op 共享底层引擎查询
- CLI 输出 Markdown 格式
- MCP 输出 JSON 格式
- `--as-context` 输出 prompt-injection-ready 的 Markdown，可直接注入 system prompt

### OpenClaw 上下文引擎

`context-engine.ts` — GBrain 作为 OpenClaw 的上下文引擎插件：
- 在每次 `assemble()` 调用时注入结构化上下文
- 包含时间、空间、操作线程信息
- 零 LLM 调用（确定性注入）
- 解决"时间扭曲"bug（压缩后的会话丢失时间感知）
