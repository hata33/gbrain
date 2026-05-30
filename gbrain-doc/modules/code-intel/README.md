# code-intel — 代码智能

> GBrain 的代码智能模块解析代码仓库，提取函数定义、调用关系和引用，
> 支持代码级的知识图谱查询。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/code-intel/recursive-walk.ts` | 递归目录遍历 |
| `src/core/code-intel/traversal-cache.ts` | 遍历缓存 |
| `src/core/code-intel/sinks/` | 代码分析输出 |

## 概述

GBrain 不仅管理文本文档，还能理解代码：

```
代码仓库
  → tree-sitter 解析 AST
  → 提取函数定义、类定义
  → 分析调用关系（caller/callee）
  → 建立代码知识图谱
  → 支持 "谁调用了这个函数？" 等查询
```

### 代码查询命令

```bash
gbrain code def <symbol>       # 查找定义
gbrain code refs <symbol>      # 查找引用
gbrain code callers <function> # 查找调用者
gbrain code callees <function> # 查找被调用者
```

### 与搜索集成

代码文件作为 `type: code` 页面存储，在搜索中通过 `source-boost` 可以调整代码结果的权重。
