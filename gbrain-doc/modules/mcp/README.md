# mcp — MCP 服务器

> GBrain 的 MCP（Model Context Protocol）服务器将大脑能力暴露给 AI Agent，
> 支持 stdio 和 HTTP 两种传输方式。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/mcp/server.ts` | MCP 服务器核心 |
| `src/mcp/tool-defs.ts` | 工具定义 |
| `src/mcp/dispatch.ts` | 请求分发 |
| `src/mcp/http-transport.ts` | HTTP 传输层 |
| `src/mcp/rate-limit.ts` | 速率限制 |

## 概述

GBrain 通过 MCP 协议向 AI Agent 暴露大脑能力：

```
AI Agent (OpenClaw / Claude / Cursor)
  ← stdio / HTTP →
GBrain MCP Server
  → BrainEngine 操作
  → 返回结果
```

### 暴露的操作

MCP 将 GBrain 的 `operations` 转换为 MCP 工具：

| MCP Tool | GBrain 操作 | 说明 |
|----------|------------|------|
| `search` | search | 混合搜索 |
| `get_page` | get_page | 获取页面 |
| `put_page` | put_page | 写入页面 |
| `recall` | recall | 事实召回 |
| `find_trajectory` | find_trajectory | 轨迹查询 |
| `list_sources` | list_sources | 列出数据源 |
| ... | ... | 更多操作 |

### 信任边界

MCP 调用者被标记为 `remote = true`（不可信），敏感操作被收紧：
- 文件上传使用严格路径限制
- synthesize/consolidate 等阶段处理器被标记为 PROTECTED
- 只有本地 CLI 调用者可以触发

### 速率限制

`rate-limit.ts` — 防止 MCP 调用者过度使用：
- 每分钟最大请求数
- 每小时最大 API 成本
- 可配置限制
