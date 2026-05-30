# engine — 概述

> BrainEngine 是 GBrain 的核心数据层抽象，定义了知识大脑与存储后端之间的完整契约。

## 设计思想

GBrain 的核心是一个"知识大脑"——它存储页面、提取实体和事实、建立知识图谱、执行混合搜索。所有这些操作都通过 BrainEngine 接口抽象，使得底层存储可以灵活切换。

```
┌─────────────────────────────────────────┐
│            GBrain 应用层                 │
│  (CLI / MCP / Autopilot / Think)        │
└──────────────────┬──────────────────────┘
                   │
         ┌─────────┴─────────┐
         │   BrainEngine     │  ← 统一接口
         │   (interface)     │
         └─────────┬─────────┘
                   │
     ┌─────────────┼─────────────┐
     │                           │
     ▼                           ▼
 PGLite Engine            Postgres Engine
 (嵌入式，零配置)         (生产级，pgvector)
```

## 两种引擎

### PGLite 引擎
- **定位**：个人使用、开发测试、零配置快速启动
- **特点**：WebAssembly 运行的 Postgres，无需服务器
- **存储**：本地文件（~/.gbrain/）
- **限制**：单进程访问，性能较低

### Postgres 引擎
- **定位**：团队使用、大规模数据、生产环境
- **特点**：完整 Postgres + pgvector 扩展
- **存储**：Supabase 或自托管 Postgres
- **优势**：并发支持、RLS 行级安全、向量索引

## 核心概念

### Page（页面）
大脑中的基本知识单元。每个页面有：
- **slug**：唯一标识（如 `people/alice`）
- **type**：页面类型（person, company, meeting, note 等 24+ 种）
- **content**：内容文本
- **frontmatter**：结构化元数据（YAML frontmatter）
- **compiled_truth**：综合后的"真相"版本
- **source_id**：所属数据源

### Source（数据源）
一个大脑可以包含多个数据源：
- `default`：默认源
- 本地文件夹（Obsidian vault、代码仓库等）
- 外部集成（邮件、日历等）

### Chunk（内容块）
页面被切分为多个 chunk 进行向量嵌入和检索：
- 每个 chunk 有独立的 embedding 向量
- 支持多模态（文本、图片）

### Edge（边/关系）
知识图谱中的关系边：
- 自动从内容中提取（works_at, invested_in, founded 等）
- 零 LLM 调用，纯规则提取
- 支持图遍历查询

## 工厂模式

`engine-factory.ts` 实现了引擎工厂：

```typescript
async function createEngine(config: EngineConfig): Promise<BrainEngine> {
  switch (config.engine) {
    case 'pglite':
      return new PGLiteEngine();
    case 'postgres':
      return new PostgresEngine();
  }
}
```

动态导入确保 PGLite 的 WASM 代码只在需要时加载，不会影响 Postgres 用户的启动速度。
