# 06 — 多脑联邦

> GBrain 支持一个用户拥有多个独立大脑，通过 BrainRegistry 路由操作，
> 实现个人脑 + 工作脑 + 团队脑的联邦架构。

## 设计思想

```
一个大脑不能满足所有需求：
  → 个人笔记（隐私）
  → 工作笔记（公司内部）
  → 团队共享（协作）

解决方案：多脑联邦
  → 每个大脑是独立的数据库
  → 通过 BrainRegistry 路由
  → 支持跨大脑搜索（latent-space 融合）
```

## 架构

```
BrainRegistry
  ├── 'host' → ~/.gbrain/data (PGLite, 个人)
  ├── 'work' → ~/work-brain (PGLite, 工作)
  └── 'team' → postgres://team-db (Postgres, 团队)

CLI: gbrain --brain work search "Alice"
MCP: brainId = "work"
```

## 大脑解析优先级

```
1. --brain <id> 参数
2. GBRAIN_BRAIN_ID 环境变量
3. .gbrain-mount dotfile（CWD 或祖先目录）
4. mounts.json 中 path 匹配 CWD 的条目
5. 默认 brain
6. 'host' 回退
```

## 团队共享

Postgres + Supabase 的 RLS 实现：
- 每个用户只能看到自己的数据
- 团队管理员可以设置共享范围
- 查询时自动过滤（RLS 策略）

## 核心代码

| 文件 | 职责 |
|------|------|
| `src/core/brain-registry.ts` | 大脑注册表 |
| `src/core/brain-resolver.ts` | 大脑解析 |
