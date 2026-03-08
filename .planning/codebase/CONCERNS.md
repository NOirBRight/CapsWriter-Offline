# Codebase Concerns

**Analysis Date:** 2026-03-08

## Security Considerations

**API Key Leakage:**
- Risk: Hardcoded OpenAI API key in backup file
- Files: `LLM/backup/轻度润色.py:14`
- Current mitigation: File is in backup directory, but still committed to git
- Recommendations: Remove API keys from source code entirely, use environment variables or config.json only

**Config File Exposure:**
- Risk: config.json may contain user API keys in production
- Files: `config.json`
- Current mitigation: api_key fields are empty in default config
- Recommendations: Add config.json to .gitignore for user-specific overrides

**UDP Broadcast:**
- Risk: UDP broadcasts can be received by any device on the local network
- Files: `util/client/udp/udp_control.py`, `config.py:116-120`
- Current mitigation: Default targets are localhost only
- Recommendations: Document security implications for LAN configurations

---

## Tech Debt

**Large Model Conversion Utility:**
- Issue: Single file with 11,433 lines handling multiple model formats
- Files: `util/fun_asr_gguf/convert_hf_to_gguf.py`
- Impact: Difficult to maintain, test, and extend
- Fix approach: Break into smaller modules by model type (LLM, Vision, Audio)

**Duplicate Hotword Modules:**
- Issue: Nearly identical code in two locations
- Files: `util/hotword/` and `util/fun_asr_gguf/hotword/`
- Impact: Bug fixes need to be applied twice, potential inconsistency
- Fix approach: Consolidate into single module, use import alias if differentiation needed

**TODO Items in Model Converter:**
- Issue: ~30+ TODO comments in convert_hf_to_gguf.py alone
- Files: `util/fun_asr_gguf/convert_hf_to_gguf.py`
- Impact: Technical debt accumulates, some items are years old
- Fix approach: Create tracking issue for converter TODOs, prioritize critical items

**Silent Config Failure:**
- Issue: JSON parse errors in config.json are silently ignored
- Files: `config.py:14-22`
- Impact: User misconfigurations go unnoticed
- Fix approach: Log warnings when config fails to load

---

## Known Bugs

**WebSocket Reconnection Loop:**
- Symptoms: Repeated "cannot connect to server" errors every 6 seconds
- Files: `util/client/websocket_manager.py:92`
- Trigger: Server process not running
- Workaround: Ensure server is started before client

**GlobalHotKey Listener Failure:**
- Symptoms: "启动 GlobalHotKeys 监听器失败" errors
- Files: `util/client/global_hotkey/global_hotkey.py:128`
- Trigger: Certain key combinations (e.g., <escape>)
- Workaround: Use different hotkey combinations

**Config Hot Reload Incomplete:**
- Issue: Some config changes require full restart
- Files: `config.py:246-255`
- Trigger: Changes to certain settings
- Workaround: Restart both client and server

---

## Performance Bottlenecks

**GGUF Model Loading:**
- Problem: Large models load slowly on first inference
- Files: `util/fun_asr_gguf/core/model_manager.py`
- Cause: Full model deserialization
- Improvement path: Implement model caching between sessions

**Hotword RAG Search:**
- Problem: Initial RAG index load is slow with large hot.txt
- Files: `util/hotword/rag_fast.py`, `util/hotword/rag_accu.py`
- Cause: Loading all hotwords into memory on startup
- Improvement path: Lazy loading or persistent index cache

**Log File Growth:**
- Problem: Log files can grow to 10MB+
- Files: `logs/client_*.log`, `logs/server_*.log`
- Cause: No log rotation implemented
- Improvement path: Add logging.handlers.RotatingFileHandler

---

## Fragile Areas

**WebSocket Connection Manager:**
- Why fragile: Complex state machine with reconnection logic
- Files: `util/client/websocket_manager.py`
- Safe modification: Test reconnection scenarios thoroughly
- Test coverage gaps: No unit tests for connection state transitions

**Audio Stream Processing:**
- Why fragile: Timing-sensitive chunk collection and streaming
- Files: `util/client/audio/stream.py`
- Safe modification: Test with various audio input devices
- Test coverage gaps: No integration tests with real microphone

**Process Lifecycle Management:**
- Why fragile: Child process spawning and cleanup is complex
- Files: `util/common/lifecycle.py`, `gui/main_window.py`
- Safe modification: Ensure all process handles are properly closed
- Test coverage gaps: No tests for process crash scenarios

---

## Code Quality Issues

**Bare Exception Handlers:**
- Issue: Many `except Exception:` without specific handling
- Files: Throughout codebase (see grep results - 100+ occurrences)
- Impact: Swallows legitimate errors, makes debugging harder
- Fix approach: Use specific exception types, add error context

**Global Mutable State:**
- Issue: Heavy use of global variables for state management
- Files: `core_client.py`, `core_server.py`
- Impact: Difficult to test components in isolation
- Fix approach: Consider dependency injection or state container

**Inconsistent Error Reporting:**
- Issue: Mix of print(), logger.error(), and silent failures
- Files: Throughout codebase
- Impact: User experience inconsistent
- Fix approach: Standardize on logging framework only

---

## Test Coverage Gaps

**No Unit Tests Directory:**
- What's not tested: Core business logic
- Files: `util/hotword/`, `util/client/`
- Risk: Silent regressions in recognition accuracy
- Priority: Medium

**No Integration Tests:**
- What's not tested: Client-server communication
- Files: WebSocket handling, UDP control
- Risk: Connection issues in production
- Priority: High

**No E2E Tests:**
- What's not tested: Full dictation workflow
- Files: Entire system
- Risk: User-facing bugs not caught
- Priority: Low (manual testing sufficient for now)

---

## Dependencies at Risk

**sherpa-onnx:**
- Risk: Version-specific, CUDA version dependent
- Impact: Breaking changes in ONNX runtime
- Migration plan: Pin versions, test with new releases

**numba:**
- Risk: JIT compilation issues on different CPU architectures
- Impact: Hotword RAG may fail on ARM Windows
- Migration plan: Add fallback to pure Python

**pyinstaller:**
- Risk: Build-time compatibility issues
- Impact: Releases may not build on new Python versions
- Migration plan: Use versioned build environment

---

## Missing Critical Features

**Backup/Restore Configuration:**
- Problem: No way to backup user config
- Blocks: User migration between machines

**Error Reporting UI:**
- Problem: Errors only in logs
- Blocks: User self-diagnosis

**Model Auto-Update:**
- Problem: Manual model download required
- Blocks: Seamless version upgrades

---

*Concerns audit: 2026-03-08*
