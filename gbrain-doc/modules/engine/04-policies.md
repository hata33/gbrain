# engine — 策略与边界

> 引擎选择策略、多源隔离、软删除策略、安全边界。

## 引擎选择策略

| 场景 | 推荐引擎 | 原因 |
|------|----------|------|
| 个人使用，< 1000 页面 | PGLite | 零配置，秒级启动 |
| 个人使用，> 1000 页面 | Postgres | 性能更好 |
| 团队共享 | Postgres + Supabase | RLS 行级安全 |
| 开发/测试 | PGLite | 隔离，不影响生产数据 |
| CI/CD | PGLite | 无需外部依赖 |

## 多源隔离

一个大脑可以包含多个数据源（source），每个源有独立的 `source_id`：

```
brain/
  ├── source: "default"    → 主 vault
  ├── source: "work"       → 工作笔记
  ├── source: "code"       → 代码仓库
  └── source: "emails"     → 邮件归档
```

### 隔离规则

- **页面 slug 在同一 source 内唯一**，不同 source 可以有同名 slug
- **搜索默认跨所有源**，可通过 `sourceId` 参数限制范围
- **模糊解析受源范围限制** — MCP 调用者只能访问被授权的源（防止 #1436 类型的信息泄露）
- **删除操作必须指定 sourceId** — 防止误删其他源的同名页面

## 软删除策略

```
softDeletePage(slug)
  → 设置 deleted_at = now()
  → 页面仍在数据库中，chunk/links 保持完整
  → 搜索结果自动排除（默认）
  → 72 小时后由 autopilot 自动清除

restorePage(slug)
  → 清除 deleted_at
  → 页面恢复可见

purgeDeletedPages(72)
  → 硬删除超过 72 小时的软删除页面
  → 级联删除所有关联数据
```

## 信任边界

GBrain 区分两种调用者：

| 调用者 | remote | 权限 |
|--------|--------|------|
| 本地 CLI | `false` | 完整权限，可执行所有操作 |
| MCP 远程调用 | `true` | 受限权限，敏感操作被收紧 |

安全敏感操作（如 `file_upload`、`synthesize`、`consolidate`）在 `remote = true` 时使用更严格的文件系统限制。三个阶段处理器（synthesize / patterns / consolidate）被标记为 PROTECTED — 只有本地调用者可以触发。

## 事务边界

- **单条写操作**：自动事务（autocommit）
- **多条写操作**：必须使用 `engine.transaction()` 包裹
- **批量操作**：调用者负责分片（如 `deletePages` 每批 ≤ 500 条）
- **原子性保证**：单条 SQL 或单个事务，全部成功或全部回滚

## 并发安全

- **Postgres**：通过连接池 + MVCC 实现并发安全
- **PGLite**：单进程访问，无并发问题
- **JSONB merge**：使用 Postgres 的 `||` 操作符，避免读-改-写竞态

## 边界情况

1. **slug 冲突**：`putPage` 使用 ON CONFLICT DO UPDATE（upsert），不报错
2. **级联删除**：`deletePage` 级联删除 chunk、links、relations、tags、raw_data、timeline、versions
3. **空大脑**：`getAllSlugs()` 返回空 Set，不报错
4. **未知 slug**：`getPage()` 返回 null，不报错
5. **软删除幂等**：重复软删除返回 null（不报错）
