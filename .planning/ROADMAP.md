# Roadmap: CapsWriter-Offline

## Overview

**Core Value:** "快、准、稳、离线" — 通过全局快捷键触发流式语音识别，松开按键后自动上屏，支持 LLM 智能润色和热词优化

**Granularity:** Coarse (3-5 phases)

---

## Phases

- [ ] **Phase 1: Core ASR Infrastructure** - Server, streaming, recognition, punctuation, config
- [ ] **Phase 2: Output & Hotword** - Keyboard simulation, toast, clipboard, hotword RAG

---

## Phase Details

### Phase 1: Core ASR Infrastructure

**Goal:** Users can trigger voice recording via CapsLock, see real-time recognition results, with proper punctuation

**Depends on:** Nothing (first phase)

**Requirements:** ASR-01, ASR-02, ASR-03, ASR-04, PUNC-01, PUNC-02, CONF-01, CONF-02, CONF-03, SERV-01, SERV-02, SERV-03

**Success Criteria** (what must be TRUE):
1. User can press CapsLock to start voice recording
2. Audio is captured and streamed to local ASR model in real-time
3. Recognition results display in real-time during recording (toast/UI)
4. Final result is submitted when user releases CapsLock
5. Punctuation model adds appropriate Chinese/English punctuation to text
6. User can configure hotkey threshold (default 0.3s) and audio segment duration (default 25s) via config.py

**Plans:** 3 plans in 3 waves

Plans:
- [ ] 01-01-PLAN.md — Test infrastructure + unit tests (hotkey, punctuation, config)
- [ ] 01-02-PLAN.md — Integration tests (streaming, server, subprocess)
- [ ] 01-03-PLAN.md — E2E tests + verification checkpoint

---

### Phase 2: Output & Hotword

**Goal:** Recognized text is typed automatically with hotword correction for accuracy

**Depends on:** Phase 1

**Requirements:** OUT-01, OUT-02, OUT-03, HOTW-01, HOTW-02

**Success Criteria** (what must be TRUE):
1. Recognized text is typed automatically via keyboard simulation at cursor position
2. Toast notification displays result as fallback when keyboard typing fails
3. Result persists in clipboard as backup
4. User can add hotwords to hot.txt file
5. Phoneme-based RAG检索 finds and applies matching hotwords to improve accuracy

**Plans:** TBD

---

## Progress Table

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Core ASR Infrastructure | 3/3 | Planning complete | - |
| 2. Output & Hotword | 0/N | Not started | - |

---

## Coverage

- **v1 Requirements:** 16 total
- **Mapped:** 16 (100%)
- **Orphaned:** 0

| Requirement | Phase |
|-------------|-------|
| ASR-01 | Phase 1 |
| ASR-02 | Phase 1 |
| ASR-03 | Phase 1 |
| ASR-04 | Phase 1 |
| PUNC-01 | Phase 1 |
| PUNC-02 | Phase 1 |
| CONF-01 | Phase 1 |
| CONF-02 | Phase 1 |
| CONF-03 | Phase 1 |
| SERV-01 | Phase 1 |
| SERV-02 | Phase 1 |
| SERV-03 | Phase 1 |
| OUT-01 | Phase 2 |
| OUT-02 | Phase 2 |
| OUT-03 | Phase 2 |
| HOTW-01 | Phase 2 |
| HOTW-02 | Phase 2 |

---

## Notes

- **v2 Requirements** (LLM, Transcription, History, Multi-model) deferred to future release
- Phase structure follows research recommendation: Core → Output
- Coarse granularity: Combined ASR integration with infrastructure (Phase 1) and output with hotword (Phase 2)

---

*Roadmap created: 2026-03-08*
