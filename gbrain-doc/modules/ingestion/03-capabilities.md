# ingestion — 能力清单

> 摄入系统的完整能力清单。

## 核心 API

| 函数 | 说明 |
|------|------|
| `importFile(path, opts)` | 导入单个文件到大脑 |
| `sync(config)` | 增量同步本地文件夹 |
| `ingest(source)` | 运行摄入守护进程 |

## IngestionSource 接口

```typescript
interface IngestionSource {
  id: string;                          // 源标识
  name: string;                        // 显示名称
  apiVersion: number;                  // API 版本

  // 生命周期
  start(ctx: IngestionSourceContext): Promise<void>;
  stop(): Promise<void>;
  health(): Promise<IngestionSourceHealth>;

  // 事件
  on(event: 'ingestion', handler: (evt: IngestionEvent) => void): void;
}
```

## IngestionEvent 类型

```typescript
type IngestionEvent = {
  type: 'create' | 'update' | 'delete';
  path: string;              // 文件路径或远程 ID
  content: string;           // 内容文本
  contentType: string;       // 内容类型（markdown, email, html...）
  metadata: Record<string, unknown>;  // 元数据
};
```

## 支持的内容类型

| 类型 | 说明 |
|------|------|
| `markdown` | Markdown 文件（Obsidian 等） |
| `email` | 邮件（Gmail、Outlook） |
| `html` | 网页内容 |
| `conversation` | 聊天记录 |
| `code` | 源代码文件 |
| `image` | 图片（多模态嵌入） |
| `pdf` | PDF 文档 |

## 测试工具

```typescript
// 摄入测试工具
import { testHarness } from 'gbrain/ingestion/test-harness';
testHarness(source)
  → 模拟摄入流程
  → 不实际写入数据库
  → 返回摄入结果供断言
```
