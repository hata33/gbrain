# cli — CLI 接口

> GBrain 的 CLI 是用户与大脑交互的主要命令行接口，
> 提供了 100+ 个子命令覆盖所有核心功能。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/cli.ts` | CLI 入口 |
| `src/commands/` | 所有子命令实现（100+ 文件） |
| `src/core/operations.ts` | 操作定义（CLI 和 MCP 共享） |

## 概述

```
gbrain
  ├── init              # 初始化大脑
  ├── import <path>     # 导入文件
  ├── sync              # 同步文件夹
  ├── search <query>    # 搜索
  ├── recall <entity>   # 记忆召回
  ├── think <query>     # 深度推理
  ├── dream             # 梦境循环（自动化维护）
  ├── autopilot         # 守护进程模式
  ├── pages             # 页面管理
  ├── sources           # 数据源管理
  ├── doctor            # 诊断与修复
  ├── eval              # 评估
  ├── serve             # 启动 MCP 服务器
  └── ...               # 更多命令
```

### 共享操作层

CLI 命令和 MCP 工具共享同一套 `operations`：
- `operations.ts` 定义所有操作
- CLI 通过 `cliOps` 映射将命令路由到操作
- MCP 通过 `buildToolDefs()` 将操作转换为 MCP 工具
- 保证 CLI 和 MCP 行为一致

### 全局参数

```bash
gbrain --brain <id> <command>    # 指定大脑
gbrain --source <id> <command>   # 指定数据源
gbrain --json <command>          # JSON 输出
gbrain --verbose <command>       # 详细日志
```
