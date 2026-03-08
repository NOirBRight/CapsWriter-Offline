# Requirements: CapsWriter-Offline

**Defined:** 2026-03-08
**Core Value:** "快、准、稳、离线" — 通过全局快捷键触发流式语音识别，松开按键后自动上屏，支持 LLM 智能润色和热词优化

## v1 Requirements

### Core Recognition

- [ ] **ASR-01**: User can trigger voice recording via global hotkey (CapsLock)
- [ ] **ASR-02**: Audio is captured and streamed to local ASR model in real-time
- [ ] **ASR-03**: Recognition results display in real-time during recording
- [ ] **ASR-04**: Final result is submitted when user releases hotkey

### Punctuation

- [ ] **PUNC-01**: Punctuation model adds appropriate punctuation to recognized text
- [ ] **PUNC-02**: Bilingual punctuation support (Chinese and English)

### Output

- [ ] **OUT-01**: Recognized text is typed automatically via keyboard simulation
- [ ] **OUT-02**: Toast notification displays result as fallback when typing fails
- [ ] **OUT-03**: Result persists in clipboard as backup

### Configuration

- [ ] **CONF-01**: User can configure via config.py
- [ ] **CONF-02**: Hotkey trigger threshold is adjustable (default 0.3s)
- [ ] **CONF-03**: Audio segment duration is configurable (default 25s)

### Server

- [ ] **SERV-01**: WebSocket server handles client connections
- [ ] **SERV-02**: ASR runs in separate subprocess (non-blocking)
- [ ] **SERV-03**: Server handles multiple audio segments with overlap

### Hotword

- [ ] **HOTW-01**: User can add hotwords to hot.txt
- [ ] **HOTW-02**: Phoneme-based RAG检索 finds matching hotwords

## v2 Requirements

### LLM Integration

- **LLM-01**: User can enable LLM polishing of recognized text
- **LLM-02**: User can select from multiple LLM providers (Ollama, OpenAI, DeepSeek, Claude)
- **LLM-03**: Role system allows different LLM behavior configurations

### Audio Transcription

- **TRAN-01**: User can drag audio file into app for transcription
- [ ] **TRAN-02**: Output includes timestamps (SRT format)

### History

- [ ] **HIST-01**: Transcriptions are archived by date
- [ ] **HIST-02**: User can browse historical transcriptions

### Models

- [ ] **MODL-01**: User can switch between ASR models (FunASR, Paraformer, SenseVoice)
- [ ] **MODL-02**: Model selection UI shows accuracy/speed tradeoffs

## Out of Scope

| Feature | Reason |
|---------|--------|
| Online ASR services | 坚持纯离线，保护隐私 |
| Mobile platforms | Windows 桌面专用 |
| Cloud sync | 保持离线优先 |
| Multi-user concurrent | 单用户本地设计 |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| ASR-01 | Phase 1 | Pending |
| ASR-02 | Phase 1 | Pending |
| ASR-03 | Phase 1 | Pending |
| ASR-04 | Phase 1 | Pending |
| PUNC-01 | Phase 1 | Pending |
| PUNC-02 | Phase 1 | Pending |
| OUT-01 | Phase 2 | Pending |
| OUT-02 | Phase 2 | Pending |
| OUT-03 | Phase 2 | Pending |
| CONF-01 | Phase 1 | Pending |
| CONF-02 | Phase 1 | Pending |
| CONF-03 | Phase 1 | Pending |
| SERV-01 | Phase 1 | Pending |
| SERV-02 | Phase 1 | Pending |
| SERV-03 | Phase 1 | Pending |
| HOTW-01 | Phase 2 | Pending |
| HOTW-02 | Phase 2 | Pending |

**Coverage:**
- v1 requirements: 16 total
- Mapped to phases: 16
- Unmapped: 0 ✓

---
*Requirements defined: 2026-03-08*
*Last updated: 2026-03-08 after initial definition*
