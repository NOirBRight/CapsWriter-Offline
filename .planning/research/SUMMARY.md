# Research Synthesis Summary

**Project:** CapsWriter-Offline
**Synthesized:** 2026-03-08

---

## Executive Summary

CapsWriter-Offline is an offline voice dictation application that processes speech entirely locally using Sherpa-ONNX ASR models. The architecture follows a proven Client-Server pattern where the Client handles UI/input and the Server subprocess handles CPU-intensive AI inference, ensuring the WebSocket heartbeat and UI remain responsive. The core value proposition is privacy-first, offline Chinese dictation with phoneme-based hotword correction that addresses a common ASR failure mode (homophone confusion).

The recommended approach prioritizes: (1) establishing C/S connectivity, (2) integrating Sherpa-ONNX streaming, (3) adding keyboard simulation and hotword correction, then (4) polishing with LLM features. Key risks center on audio pipeline blocking, hotkey/IME conflicts, and model loading memory spikes. All have documented mitigations.

---

## Key Findings

### From STACK.md

- **Python 3.10-3.12** is the standard runtime with best package compatibility
- **Sherpa-ONNX** is the recommended ASR engine: native ONNX support, no PyTorch at runtime, streaming support, active development
- **sounddevice** replaces pyaudio (fewer Windows binary issues, async-native)
- **pynput + keyboard** combo handles CapsLock hotkey and text output without IME conflicts
- **PyInstaller 6.0+** is the industry standard for Windows .exe builds with CUDA support
- Key dependencies: sounddevice, numpy, numba (JIT), websockets, tkinter, ttkbootstrap, pystray

### From FEATURES.md

- **Table Stakes (P0):** Offline ASR, real-time streaming, global hotkey (CapsLock), keyboard text input, punctuation, Chinese language support, audio capture, basic config UI
- **Differentiators (P1):** Phoneme-based hotword RAG (key differentiator), LLM polishing, role-based agents, audio file transcription, history archive, multi-model support
- **Anti-Features:** Online ASR, mobile app, cloud sync, plugin architecture - all explicitly avoided to maintain focus

### From ARCHITECTURE.md

- **C/S Architecture:** Client (UI/input) + Server (ASR subprocess) connected via WebSocket
- **Data Flow:** CapsLock press → audio capture (25s chunks, 2s overlap) → WebSocket stream → Sherpa-ONNX → punctuation → text output via keyboard simulation
- **Key Patterns:** Process separation for CPU-bound work, dual result text merging (text vs text_accu), chunked streaming with overlap, two-stage hotword retrieval (FastRAG + AccuRAG)
- **Build Order:** Core Infrastructure → ASR Integration → Output & Polish → Features & Polish

### From PITFALLS.md

- **Critical:** Audio pipeline blocking main thread (mitigation: async thread with queue), text insertion at wrong position (save/restore window handle), hotkey conflicts with IME (configurable hotkey + IME detection), model loading memory spikes (sequential loading + model pooling), WebSocket instability (heartbeat + reconnection)
- **Moderate:** Process race conditions (server ready signal), audio format mismatch (explicit 16kHz/mono config), PyInstaller model loading (_MEIPASS paths), Chinese IME compatibility (IME state detection), LLM API failures (timeout + fallback)
- **Minor:** Inefficient hotword search (pre-indexed RAG), logging without rotation (RotatingFileHandler)

---

## Implications for Roadmap

### Suggested Phase Structure

1. **Phase 1: Core Infrastructure**
   - Client skeleton (hotkey detection, audio capture)
   - Server WebSocket echo
   - Basic connectivity test
   - **Why first:** No communication = nothing works. Establish C/S before adding AI.

2. **Phase 2: ASR Integration**
   - Sherpa-ONNX in Server subprocess
   - Streaming audio pipeline (25s chunks, 2s overlap)
   - Punctuation model
   - **Why second:** Core value proposition. Get working dictation before polish.

3. **Phase 3: Output & Polish**
   - Keyboard simulation (window handle save/restore)
   - Hotword system (FastRAG + AccuRAG)
   - Toast display UI
   - Basic LLM integration (with timeout/fallback)
   - **Why third:** Dictation works, now make it usable and accurate.

4. **Phase 4: Features & Polish**
   - Role system (multi-agent configuration)
   - Audio file transcription (drag-drop workflow)
   - History/diary archive
   - Multi-model support
   - **Why last:** Nice-to-have enhancements built on working core.

### Research Flags

| Phase | Needs Research | Standard Patterns |
|-------|----------------|-------------------|
| Phase 1 | No | WebSocket + threading well-documented |
| Phase 2 | Maybe | Sherpa-ONNX streaming patterns need verification |
| Phase 3 | Yes | LLM polishing UX patterns need validation |
| Phase 4 | No | Transcription/history are straightforward |

### Pitfalls to Avoid Per Phase

- **Phase 1-2:** Audio pipeline blocking, WebSocket drops, process race conditions
- **Phase 2-3:** Text at wrong position, IME conflicts, audio format mismatch
- **Phase 3-4:** PyInstaller model loading, LLM API failures, logging without rotation

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Sherpa-ONNX, sounddevice, PyInstaller proven in production |
| Features | MEDIUM | Table stakes clear; differentiator value needs market validation |
| Architecture | HIGH | C/S pattern well-established; codebase already uses it |
| Pitfalls | HIGH | Comprehensive coverage based on domain expertise |

### Gaps to Address

- **Version pinning:** requirements*.txt should pin exact versions for reproducibility
- **CUDA optionality:** Server requires CUDA build; CPU-only fallback needs testing
- **LLM polish UX:** How users interact with LLM-improved text needs validation
- **IME compatibility:** Test with Sogou, Baidu, Microsoft Pinyin IMEs

---

## Sources

- STACK.md: Project codebase analysis, Sherpa-ONNX documentation, requirements files
- FEATURES.md: Project CLAUDE.md, PROJECT.md, domain expertise
- ARCHITECTURE.md: Project CLAUDE.md, PROJECT.md, Sherpa-ONNX docs, codebase
- PITFALLS.md: Domain expertise in offline ASR, Windows input simulation, PyInstaller patterns
