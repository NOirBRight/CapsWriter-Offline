# CapsWriter-Offline

## What This Is

离线语音识别输入工具 (Offline Voice Dictation Software) — 全本地 AI 模型运行，保护隐私。为 Windows 用户提供 CapsLock 快捷键触发的实时语音转文字输入，支持中英文。

## Core Value

**"快、准、稳、离线"** — 通过全局快捷键触发流式语音识别，松开按键后自动上屏，支持 LLM 智能润色和热词优化。

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] 语音识别 (ASR) - 离线模型 (FunASR, Paraformer, SenseVoice)
- [ ] 标点生成 - Punct-CT-Transformer 离线标点模型
- [ ] 实时流式识别 - WebSocket 客户端/服务端架构
- [ ] 全局快捷键监听 - CapsLock 触发录音
- [ ] 热词系统 - 音素 RAG 两阶段检索
- [ ] LLM 润色 - 多提供商支持 (Ollama, OpenAI, DeepSeek, Claude, etc.)
- [ ] 角色系统 - 模块化 LLM 角色配置
- [ ] 键盘模拟输入 - 自动上屏
- [ ] 音频转录 - 拖入音频文件生成 .srt
- [ ] 历史归档 - 按日期保存录音和转录

### Out of Scope

- 在线 ASR 服务 — 坚持纯离线
- 移动端支持 — Windows 桌面专用

## Context

- **架构**: C/S 架构，Server 处理 WebSocket + 独立子进程运行 AI 模型
- **技术栈**: Python, PyInstaller, Sherpa-ONNX, WebSocket, Tkinter
- **模型**: FunASR-Nano, Paraformer, SenseVoice, Punct-CT-Transformer
- **用户语言**: 中文 (Chinese)
- **系统**: Windows 10

## Constraints

- **离线优先**: 所有 AI 模型本地运行，保护隐私
- **Windows only**: 打包为 .exe，使用 PyInstaller
- **Python 环境**: conda activate capswriter

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| C/S 架构分离推理 | CPU 密集的推理不阻塞 WebSocket 心跳 | — Pending |
| 音素 RAG 热词 | 解决中文同音字问题 | — Pending |

---
*Last updated: 2026-03-08 after initialization*
