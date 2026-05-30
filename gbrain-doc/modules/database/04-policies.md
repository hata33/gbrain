# database — 策略与边界

> 数据库引擎的配置策略、并发控制、数据安全和边界情况。

## 引擎选择决策树

```
需要团队共享？
  → 是 → Postgres + Supabase（RLS 行级安全）
  → 否 → 数据量 > 1000 页面？
    → 是 → Postgres（性能更好）
    → 否 → PGLite（零配置）
```

## 连接池策略（Postgres）

| 配置 | 默认值 | 说明 |
|------|--------|------|
| `poolSize` | 10 | 最大连接数 |
| `prepare` | auto | 准备语句（PgBouncer 端口自动禁用） |
| `connect_timeout` | 30s | 连接超时 |

### PgBouncer 兼容

端口 6543 自动识别为 PgBouncer 事务模式：
- 禁用准备语句（`prepare: false`）
- 避免 "prepared statement does not exist" 错误
- 可通过 `GBRAIN_PREPARE=true` 覆盖

## 文件锁策略（PGLite）

```
PGLite 启动
  → pglite-lock.ts 尝试获取文件锁
  → 锁已被占用 → 报错 "另一个 gbrain 进程正在运行"
  → 锁获取成功 → 正常运行
  → 进程退出/崩溃 → 自动释放锁
```

## Schema 迁移策略

- **向前兼容**：新版本代码可以读取旧版本 Schema
- **向后不兼容**：旧版本代码不能读取新版本 Schema
- **迁移时机**：`gbrain upgrade` 自动触发，或手动 `gbrain apply-migrations --yes`
- **备份**：迁移前自动备份（Postgres 通过 pg_dump，PGLite 通过文件复制）

## 数据安全

1. **事务原子性**：所有写操作在事务内完成
2. **级联删除**：删除页面时自动清理 chunk、links、relations
3. **软删除保护**：72 小时缓冲期，可恢复
4. **连接字符串**：不记录到日志（敏感信息过滤）

## 边界情况

1. **数据库不存在**：PGLite 自动创建，Postgres 报错
2. **Schema 版本不匹配**：报错并提示运行 `apply-migrations`
3. **pgvector 扩展缺失**：Postgres 启动时报错，提示安装
4. **磁盘空间不足**：写操作失败，错误信息包含磁盘状态
5. **连接超时**：自动重试（指数退避），超过最大重试次数后报错
6. **并发写入冲突**：Postgres 通过 MVCC 自动处理，PGLite 通过文件锁串行化
