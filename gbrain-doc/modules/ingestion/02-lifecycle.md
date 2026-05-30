# ingestion — 生命周期

> 内容从源到大脑的完整摄入流程。

## 摄入流程

```
1. 源扫描
   → 遍历源中的所有文件/条目
   → 计算内容哈希
   → 检查是否需要更新（与上次同步对比）
  │
  ▼
2. 去重检查
   → findDuplicatePage(sourceId, { hash, frontmatterId })
   → 已存在且内容未变 → 跳过
   → 已存在但内容变化 → 更新
   → 不存在 → 新建
  │
  ▼
3. 内容解析
   → 解析 Markdown frontmatter
   → 提取元数据（标题、标签、日期、类型）
   → 推断页面类型（person, company, meeting...）
  │
  ▼
4. 内容切块
   → 按段落/标题切分为 chunks
   → 每个 chunk 保留上下文信息
   → 控制 chunk 大小（token 预算）
  │
  ▼
5. 向量嵌入
   → embed(chunks) 生成嵌入向量
   → 存储到 content_chunks 表
  │
  ▼
6. 页面写入
   → putPage(slug, { type, content, frontmatter... })
   → 级联写入 chunks、links
  │
  ▼
7. 后处理
   → 提取实体和事实（extract 阶段）
   → 建立链接关系（reconcile-links）
   → 更新知识图谱边
   → 记录摄入日志
```

## 增量同步

```
gbrain sync
  → 检测文件变更（git diff / 文件系统监控）
  → 只处理变更的文件
  → 删除已移除的文件（软删除）
  → 更新已修改的文件
  → 新增新文件
```

## 守护进程模式

`daemon.ts` — 持续运行的摄入服务：
```
gbrain ingest
  → 启动摄入守护进程
  → 监听源变更（文件系统事件、Webhook）
  → 实时摄入新内容
  → 记录健康状态
```
