# Feature Landscape

**Project:** CapsWriter-Offline
**Domain:** Offline Voice Dictation Software
**Researched:** 2026-03-08
**Confidence:** LOW (web search unavailable; based on project context + training data)

## Executive Summary

Offline voice dictation software differs from cloud-based solutions (Dragon, Google Speech, Whisper API) in one critical way: **all processing happens locally**. This creates unique feature requirements around model performance, resource usage, and privacy. The feature landscape divides into table stakes (must-have for viability), differentiators (competitive advantage), and anti-features (explicitly avoided).

## Table Stakes

Features users expect. Missing these = product feels incomplete or unusable.

| Feature | Why Expected | Complexity | Dependencies |
|---------|--------------|------------|--------------|
| **Offline ASR** | Core value proposition - no internet required | High | Sherpa-ONNX, model files |
| **Real-time streaming** | Dictation must feel responsive; batch processing unacceptable | High | WebSocket, chunked audio |
| **Global hotkey trigger** | Must work in any application | Medium | pynput, OS-level hooks |
| **Keyboard text input** | Natural dictation flow - types into active app | Medium | keyboard/pynput simulation |
| **Punctuation generation** | Raw ASR output lacks punctuation; unusable for sentences | Medium | Punct-CT-Transformer |
| **Chinese language support** | Primary user base | High | FunASR/Paraformer models |
| **Audio capture** | Must work with system microphone | Low | sounddevice |
| **Basic config UI** | Users need to select device, adjust sensitivity | Low | tkinter |

### Minimum Viable Dictation Flow

```
Hotkey Press → Start Recording → Chunk Audio → Stream to Server →
ASR Processing → Punctuation → Return Text → Hotkey Release →
Simulate Keyboard Input → Text Appears in Active App
```

## Differentiators

Features that set CapsWriter-Offline apart from alternatives. Not expected, but valued.

| Feature | Value Proposition | Complexity | Dependencies |
|---------|-------------------|------------|--------------|
| **Phoneme-based hotword** | Fixes Chinese homophone errors (e.g., "shi" vs "xi") | High | pypinyin, RAG retrieval |
| **LLM polishing** | Raw ASR output improved by local LLM | Medium | Ollama/local LLM |
| **Role-based LLM agents** | Context-aware responses (translation, code, assistant) | Medium | LLM provider abstraction |
| **Audio file transcription** | Convert long recordings to text | Low | ffmpeg, SRT generation |
| **History/diary archive** | Searchable audio + transcript history | Low | File I/O, markdown |
| **Multi-model support** | User chooses accuracy vs speed tradeoff | Medium | Model switching logic |
| **Toast/display UI** | Shows result before typing; allows edit | Low | Tkinter overlay |
| **Hot-reload config** | Edit hotwords/roles without restart | Low | watchdog |
| **Multi-language ASR** | SenseVoice for ja/ko/yue | Medium | Model selection |

### Hotword System (Key Differentiator)

The phoneme-based RAG hotword system is the strongest differentiator:

1. **Problem:** Chinese has 400+ pinyin syllables but only ~1,200 common characters. ASR frequently outputs wrong character.
2. **Solution:** Phoneme RAG retrieves similar-sounding alternatives from user-defined hotwords.
3. **Why differentiated:** Most offline tools lack sophisticated hotword systems.

## Anti-Features

Features explicitly NOT built. Clear rationale for exclusion.

| Anti-Feature | Why Avoid | What to Do Instead |
|--------------|-----------|-------------------|
| **Online ASR services** | Violates privacy-first core value | Keep pure offline |
| **Mobile app** | Complexity explosion; different UX patterns | Focus on Windows desktop |
| **Cloud sync** | Privacy violation | Local history only |
| **Plugin/extension architecture** | Maintenance burden | Configurable via files |
| **Multiple hotkey方案** | UI complexity; CapsLock is ergonomic | Document alternatives in wiki |
| **Video processing** | Beyond scope | Stick to audio |
| **Speaker diarization** | Not needed for dictation | Single speaker assumed |
| **Enterprise server deployment** | Different product | Document user-run option |

## Feature Dependencies

```
ASR Engine
    └── Punctuation Model
            └── Streaming Pipeline
                    ├── Hotkey Trigger (pynput)
                    ├── Audio Capture (sounddevice)
                    └── Text Output (keyboard simulation)

LLM Polishing (optional, parallel)
    ├── Role System
    │       ├── Context Assembly (history, selection, hotwords)
    │       └── Output Mode (typing/toast)
    └── Hotword RAG
            ├── FastRAG (Numba JIT)
            └── AccuRAG (phoneme matching)
```

## MVP Recommendation

### Ship First (P0)

1. **Offline ASR** - Core value; cannot launch without
2. **Global hotkey (CapsLock)** - Primary trigger mechanism
3. **Keyboard text input** - Primary output method
4. **Basic config** - Device selection, threshold settings
5. **Chinese punctuation** - Makes output readable

### Add Next (P1)

6. **Hotword system (phoneme RAG)** - Strong differentiator
7. **Toast display** - Shows result before typing
8. **History archive** - Searchable transcripts
9. **Model switching** - FunASR vs Paraformer vs SenseVoice

### Polish Later (P2)

10. **LLM polishing** - Requires Ollama setup
11. **Role system** - Multi-agent configuration
12. **Audio file transcription** - Drag-drop workflow
13. **Multi-language** - Japanese, Korean, Cantonese

## Feature Complexity Notes

### High Complexity (require significant dev time)

- **Phoneme RAG hotwords**: Two-stage retrieval, Numba JIT optimization, fuzzy matching thresholds
- **Streaming pipeline**: WebSocket architecture, chunked processing, dual result types (text vs text_accu)
- **LLM integration**: Provider abstraction, context assembly, async streaming

### Medium Complexity (straightforward implementation)

- **Hotkey trigger**: pynput handles most cases
- **Model switching**: File path changes
- **History/diary**: File I/O + markdown

### Low Complexity (can ship quickly)

- **Config UI**: Standard tkinter forms
- **Audio capture**: sounddevice is well-documented
- **Toast display**: Tkinter overlay

## Sources

- Project CLAUDE.md (existing feature set)
- Project PROJECT.md (requirements)
- Domain knowledge: offline ASR tools (Whisper Desktop, Dragon Offline, Speech Note)
- Note: Web search unavailable for verification; confidence LOW
