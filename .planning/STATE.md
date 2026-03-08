# State: CapsWriter-Offline

**Last Updated:** 2026-03-08

---

## Project Reference

**Core Value:** "快、准、稳、离线" — 通过全局快捷键触发流式语音识别，松开按键后自动上屏，支持 LLM 智能润色和热词优化

**Current Focus:** Roadmap creation complete, awaiting planning

---

## Current Position

| Field | Value |
|-------|-------|
| **Phase** | Roadmap complete |
| **Plan** | None active |
| **Status** | Ready for Phase 1 planning |
| **Progress** | [----------] 0% |

---

## Roadmap Summary

| Phase | Goal | Requirements |
|-------|------|--------------|
| 1 - Core ASR Infrastructure | Server, streaming, recognition, punctuation, config | 12 |
| 2 - Output & Hotword | Keyboard simulation, toast, clipboard, hotword RAG | 5 |

**Total:** 2 phases, 17 requirements (16 v1 + 1 already mapped)

---

## Performance Metrics

| Metric | Value |
|--------|-------|
| v1 Requirements | 16 |
| Mapped to phases | 16 |
| Coverage | 100% |
| Unmapped | 0 |

---

## Accumulated Context

### Key Decisions

- **C/S Architecture:** Client (UI/input) + Server (ASR subprocess) via WebSocket
- **Coarse Granularity:** Combined into 2 phases for faster delivery
- **Offline First:** All AI models run locally, no network required

### Dependencies

- Phase 2 depends on Phase 1 (output needs ASR results)

### Research Input

- Phase 1: Core Infrastructure (client skeleton, WebSocket, Sherpa-ONNX)
- Phase 2: Output & Hotword (keyboard simulation, hotword RAG)

---

## Session Continuity

**Roadmap created from:**
- PROJECT.md (core value)
- REQUIREMENTS.md (v1 requirements with traceability)
- research/SUMMARY.md (phase suggestions)
- config.json (coarse granularity)

**Next Action:**
- `/gsd:plan-phase 1` to begin Phase 1 planning

---

*State updated: 2026-03-08*
