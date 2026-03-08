# Domain Pitfalls

**Project:** CapsWriter-Offline
**Domain:** Offline Voice Dictation Software
**Researched:** 2026-03-08

---

## Critical Pitfalls

Mistakes that cause rewrites or major issues.

### Pitfall 1: Audio Pipeline Blocking Main Thread

**What goes wrong:** Recording and audio processing blocks the main/GUI thread, causing UI freeze, hotkey unresponsiveness, and system instability.

**Why it happens:** Using synchronous audio capture (e.g., `pyaudio` stream without threading) in the main process. The Sherpa-ONNX model inference in the same process also blocks.

**Consequences:**
- User presses CapsLock but nothing happens (appears laggy)
- GUI becomes unresponsive during long recordings
- Hotkey events get dropped or delayed

**Prevention:** Use a dedicated audio capture thread with a queue. The architecture already uses separate processes (Server/Client), but ensure the Client's audio capture loop is fully asynchronous.

**Detection:**
- Check if `logs/client.log` shows audio callback delays
- Monitor UI responsiveness during recording
- Test with long recordings (>10 seconds)

**Phase:** Core Recognition Pipeline (Phase 1-2)

---

### Pitfall 2: Text Insertion at Wrong Cursor Position

**What goes wrong:** Simulated keyboard typing inserts text at wrong position, overwriting existing text, or appearing in wrong application.

**Why it happens:**
- Cursor position changes between recognition start and text output
- Target application doesn't have focus when typing simulation fires
- Background application receives keystrokes instead

**Consequences:**
- User frustration: text appears in wrong place
- Corrupts existing document content
- Forces user to manually correct

**Prevention:**
- Save and restore window handle before typing
- Add small delay (100-200ms) after window activation
- Implement "typing" mode vs "toast" mode as fallback
- Log target window title for debugging

**Phase:** Input Simulation & UI (Phase 2-3)

---

### Pitfall 3: Hotkey Conflicts with System Applications

**What goes wrong:** CapsLock (or other chosen hotkey) conflicts with system shortcuts, other software, or IME (Input Method Editor) behavior.

**Why it happens:**
- CapsLock is used by some IME systems (Chinese input methods)
- Some applications intercept CapsLock for their own use
- Global hotkey registration fails silently on permission issues

**Consequences:**
- Hotkey doesn't trigger consistently
- Unexpected behavior in other applications
- Poor user experience

**Prevention:**
- Provide configurable hotkey in `config.py`
- Detect IME state and handle gracefully
- Log hotkey registration failures prominently
- Test with common Chinese IMEs (Sogou, Baidu, Microsoft Pinyin)

**Phase:** Global Hotkey System (Phase 1-2)

---

### Pitfall 4: Model Loading Memory Spikes

**What goes wrong:** Loading Sherpa-ONNX models causes massive memory spike (1-2GB), triggering OOM or system sluggishness.

**Why it happens:**
- Loading multiple models simultaneously (ASR + Punctuation)
- No memory management between inference calls
- Models not unloaded when not in use

**Consequences:**
- System becomes unusable during model load
- OOM kills the application
- Other applications crash

**Prevention:**
- Load models sequentially, not simultaneously
- Implement model pooling (keep model in memory but limit concurrent uses)
- Add memory monitoring and logging
- Consider model quantization (int8) for reduced memory

**Phase:** Server Architecture (Phase 2)

---

### Pitfall 5: WebSocket Connection Instability

**What goes wrong:** WebSocket connection drops, reconnects fail, or messages get lost during recognition.

**Why it happens:**
- Server process crash or restart
- Network/processing latency causes timeout
- No heartbeat/ping-pong mechanism
- Client doesn't handle reconnection gracefully

**Consequences:**
- Recognition results lost mid-stream
- User has to re-record
- "Connection refused" errors confuse users

**Prevention:**
- Implement WebSocket heartbeat (ping/pong) every 5-10 seconds
- Add reconnection logic with exponential backoff
- Queue audio chunks during reconnection
- Show clear connection status in UI

**Phase:** Real-time Streaming (Phase 2)

---

## Moderate Pitfalls

### Pitfall 6: Cross-Process Communication Race Conditions

**What goes wrong:** Server and Client processes desync, especially during startup or shutdown.

**Why it happens:**
- Server not ready when Client connects
- Client starts before Server finishes loading models
- No proper process lifecycle management

**Consequences:**
- "Connection refused" errors on startup
- Race condition between recognition trigger and server ready
- Zombie Server processes after Client crash

**Prevention:**
- Implement Server ready signal before Client attempts connection
- Add startup timeout with clear error messages
- Use proper process group management for clean shutdown
- Implement health check endpoint on Server

**Phase:** C/S Architecture (Phase 1-2)

---

### Pitfall 7: Audio Format Mismatch

**What goes wrong:** Audio captured by Client doesn't match Server/model expectations, causing recognition failures or garbage output.

**Why it happens:**
- Microphone sample rate differs (44100 vs 16000)
- Bit depth mismatch (16-bit vs 32-bit)
- Channel configuration (stereo vs mono)

**Consequences:**
- Empty recognition results
- Extremely slow recognition
- Distorted output text

**Prevention:**
- Explicitly configure audio capture (16kHz, 16-bit, mono)
- Resample audio in Client before sending
- Log audio format for debugging
- Add audio validation in Server

**Phase:** Audio Pipeline (Phase 1-2)

---

### Pitfall 8: PyInstaller Model Loading at Runtime

**What goes wrong:** PyInstaller-packed .exe cannot find or load model files (`models/` directory).

**Why it happens:**
- Models not included in PyInstaller spec
- Path resolution fails when running from different directory
- `_MEIPASS` temp extraction issues on Windows

**Consequences:**
- Application starts but recognition fails immediately
- "Model not found" errors
- Works in dev but fails in packaged build

**Prevention:**
- Use `--add-data` to include entire `models/` folder
- Use `sys._MEIPASS` for path resolution in packaged app
- Test packaged build on clean Windows system
- Include model verification on startup

**Phase:** Build & Deployment (Phase 3-4)

---

### Pitfall 9: Chinese IME Compatibility

**What goes wrong:** Text output doesn't work correctly when Chinese IME is active, or IME intercepts hotkey.

**Why it happens:**
- Keyboard simulation doesn't account for IME state
- Some IME require specific input method (IME_SETACTIVE)
- Text gets pre-converted by IME before application receives it

**Consequences:**
- Recognition text appears as pinyin, not Chinese characters
- IME composition window interferes with toast display
- Hotkey doesn't work when IME is enabled

**Prevention:**
- Detect IME state before typing simulation
- Optionally disable IME during output
- Use `pywinauto` or `win32api` for proper IME handling
- Test with Sogou, Baidu, Microsoft Pinyin IME

**Phase:** Input Simulation (Phase 2-3)

---

### Pitfall 10: LLM Provider API Failures

**What goes wrong:** LLM polishing fails silently or crashes the application when API is unavailable.

**Why it happens:**
- Ollama not running
- Network timeout
- Invalid API key
- Rate limiting

**Consequences:**
- Application crashes during polish step
- User waits indefinitely for "polishing" that will never complete
- No fallback when LLM fails

**Prevention:**
- Add timeout to all LLM API calls (10-30 seconds)
- Implement retry with exponential backoff (max 3 retries)
- Fallback to original text if LLM fails
- Show clear error message in UI
- Log full error for debugging

**Phase:** LLM Integration (Phase 3-4)

---

## Minor Pitfalls

### Pitfall 11: Inefficient Hotword RAG Search

**What goes wrong:** Hotword search takes too long, delaying text output.

**Why it happens:**
- Linear search through large hotword list
- No indexing
- Python GIL limits parallelism

**Prevention:**
- Use pre-computed phoneme index (already implemented with FastRAG)
- Limit hotword list size (<10000 entries)
- Use Numba JIT for fuzzy matching

**Phase:** Hotword System (Phase 3)

---

### Pitfall 12: Logging Without Rotation

**What goes wrong:** Log files grow infinitely, consuming disk space.

**Why it happens:**
- No log rotation configured
- Debug logging left enabled in production
- No log size limit

**Prevention:**
- Use `logging.handlers.RotatingFileHandler`
- Set max file size (10MB) and backup count (3)
- Separate client.log and server.log

**Phase:** Build & Deployment (Phase 4)

---

## Phase-Specific Warnings

| Phase Topic | Likely Pitfall | Mitigation |
|-------------|---------------|------------|
| Core Recognition | Audio pipeline blocking | Async thread with queue |
| C/S Architecture | Process race conditions | Server ready signal + health check |
| Real-time Streaming | WebSocket drops | Heartbeat + reconnection |
| Input Simulation | Wrong cursor position | Window handle save/restore |
| LLM Integration | API failures | Timeout + fallback |
| Build/Packaging | Model loading fails | Test packaged build early |
| Hotword System | Search too slow | Pre-indexed phoneme RAG |

---

## Sources

- Domain expertise in offline voice dictation software (Sherpa-ONNX, FunASR, Paraformer)
- Common patterns in Python audio applications (PyAudio, sounddevice)
- Windows input simulation best practices (pywinauto, pynput)
- PyInstaller packaging patterns for ML models
