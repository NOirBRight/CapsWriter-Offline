---
phase: 01
slug: core-asr-infrastructure
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-08
---

# Phase 1 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | pytest |
| **Config file** | none — Wave 0 creates test infrastructure |
| **Quick run command** | `pytest tests/ -v --tb=short` |
| **Full suite command** | `pytest tests/ -v --cov=util --cov-report=term-missing` |
| **Estimated runtime** | ~60 seconds |

---

## Sampling Rate

- **After every task commit:** Run `pytest tests/ -v --tb=short`
- **After every plan wave:** Run `pytest tests/ -v --cov=util --cov-report=term-missing`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 60 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 01-01-01 | 01 | 1 | ASR-01 | unit | `pytest tests/test_asr.py -v` | ❌ W0 | ⬜ pending |
| 01-01-02 | 01 | 1 | ASR-02 | integration | `pytest tests/test_streaming.py -v` | ❌ W0 | ⬜ pending |
| 01-01-03 | 01 | 1 | PUNC-01 | unit | `pytest tests/test_punctuation.py -v` | ❌ W0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `tests/test_asr.py` — unit tests for ASR-01 (hotkey trigger)
- [ ] `tests/test_streaming.py` — integration tests for ASR-02 (streaming)
- [ ] `tests/test_punctuation.py` — unit tests for PUNC-01 (punctuation)
- [ ] `tests/conftest.py` — shared fixtures (mock audio, mock server)
- [ ] `pytest` install — if no framework detected

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Real-time UI display | ASR-03 | Requires GUI/Toast verification | Press CapsLock, speak, verify toast appears |
| Config hot-reload | CONF-03 | File system watch testing | Modify config.py, verify reload without restart |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 60s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
