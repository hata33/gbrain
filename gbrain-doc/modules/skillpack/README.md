# skillpack — 技能包

> GBrain 的技能包系统允许用户将自定义技能（数据源、处理器、命令）打包发布，
> 通过 ClawHub 或 npm 分享给社区。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/skillpack/installer.ts` | 技能包安装器 |
| `src/core/skillpack/harvest.ts` | 技能包收割 |
| `src/core/skillpack/bundle.ts` | 技能包打包 |
| `src/core/skillpack/doctor.ts` | 技能包诊断 |
| `src/core/skillpack/endorse.ts` | 技能包背书 |
| `src/core/skillpack/pack-publish.ts` | 发布工具 |

## 概述

技能包是 GBrain 的扩展机制：

```
skillpack/
  ├── manifest.yaml      # 技能包清单
  ├── skills/            # 技能文件
  ├── sources/           # 自定义数据源
  ├── handlers/          # 事件处理器
  └── templates/         # 模板文件
```

### 技能包类型

| 类型 | 说明 |
|------|------|
| `source` | 自定义数据源（如 Slack、Notion） |
| `skill` | 自定义技能（如富化、分析） |
| `template` | 页面模板 |
| `handler` | 事件处理器 |

### 发布流程

```bash
gbrain skillpack init        # 初始化技能包
gbrain skillpack harvest     # 收割现有技能
gbrain skillpack bundle      # 打包
gbrain skillpack publish     # 发布到 ClawHub
```

### 安装流程

```bash
gbrain skillify <skillpack>  # 安装技能包
  → 下载并验证
  → 安装到 skills/ 目录
  → 注册到配置
  → 生效
```
