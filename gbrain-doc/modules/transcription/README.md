# transcription — 语音转录

> GBrain 的转录服务将音频文件转换为文字，支持大文件分段处理和多 Provider。

## 核心源码

| 文件 | 职责 |
|------|------|
| `src/core/transcription.ts` | 转录服务 |
| `src/core/transcripts.ts` | 转录管理 |
| `src/core/diarize/` | 说话人分离 |

## 概述

```
音频文件（会议录音、语音笔记...）
  → 文件大小检查
  → > 25MB → ffmpeg 分段
  → 每段调用 Whisper API
  → 合并转录结果
  → 可选：说话人分离（diarize）
  → 写入大脑页面
```

### Provider

| Provider | 说明 | 速度 |
|----------|------|------|
| Groq Whisper | 默认，快速便宜 | 快 |
| OpenAI Whisper | 备选 | 中 |

### 大文件处理

```
文件 > 25MB
  → ffmpeg 切分为 < 25MB 的片段
  → 每段独立转录
  → 拼接结果
  → 保持时间戳连续性
```

### 说话人分离

`diarize/` — 识别不同说话人：
- 基于音频特征的说话人聚类
- 标注每段话的说话人
- 用于会议记录的多人对话场景
