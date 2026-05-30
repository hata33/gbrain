# storage — 存储分层

> GBrain 的存储系统管理二进制文件（图片、音频、文档等）的存储，
> 支持本地文件系统和云端（S3、Supabase Storage）两种后端。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/storage/local.ts` | 本地文件存储 |
| `src/core/storage/s3.ts` | S3 存储 |
| `src/core/storage/supabase.ts` | Supabase Storage |
| `src/core/storage.ts` | 存储入口 |
| `src/core/storage-config.ts` | 存储配置 |

## 概述

GBrain 存储两类数据：
1. **结构化数据** — 页面、chunk、边、事实（在数据库中）
2. **二进制文件** — 图片、音频、PDF（在存储系统中）

```
文件上传
  → storage.upload(file)
  → 存储到本地/S3/Supabase
  → 返回 storage_path
  → 元数据写入 files 表
  → 页面引用 storage_path
```

### 存储后端

| 后端 | 说明 | 适用场景 |
|------|------|----------|
| local | 本地文件系统 | 个人使用 |
| s3 | AWS S3 / MinIO | 生产环境 |
| supabase | Supabase Storage | Supabase 集成 |

### 存储分层

```
热数据 → 本地 SSD（快速访问）
温数据 → S3 标准存储
冷数据 → S3 低频访问 / 归档
```
