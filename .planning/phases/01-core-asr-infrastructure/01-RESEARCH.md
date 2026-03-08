# Phase 1: Core ASR Infrastructure - Research

**Researched:** 2026-03-08
**Domain:** Offline Speech Recognition (ASR), WebSocket Streaming, Global Hotkey
**Confidence:** HIGH

## Summary

Phase 1 implements the core ASR infrastructure for CapsWriter-Offline. The project already has substantial existing code covering most requirements: Sherpa-ONNX integration for offline ASR, WebSocket server/client architecture, global hotkey detection via pynput, and Punct-CT-Transformer for punctuation.

**Primary recommendation:** Leverage existing code in `util/server/` and `util/client/` modules. The main implementation work involves integrating existing components to meet Phase 1 requirements (ASR-01 through ASR-04, PUNC-01/02, CONF-01/02/03, SERV-01/02/03).

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions
- Use Sherpa-ONNX for offline ASR (already integrated)
- Support multiple models: FunASR-Nano, Paraformer, SenseVoice
- Streaming 25s segments with 2s overlap
- WebSocket server in `util/server/`
- Punct-CT-Transformer for punctuation
- config.py in root directory
- Hotkey threshold: 0.3s default
- Audio segment: 25s default
- Dual result merging: text (simple) and text_accu (token timestamp deduplication)

### Claude's Discretion
- Exact real-time display implementation (toast vs UI)
- Model selection defaults
- Error handling specifics

### Deferred Ideas (OUT OF SCOPE)
None — discussion stayed within phase scope
</user_constraints>

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| ASR-01 | User can trigger voice recording via global hotkey (CapsLock) | Existing global_hotkey.py using pynput |
| ASR-02 | Audio is captured and streamed to local ASR model in real-time | Existing recorder.py and server_ws_recv.py with WebSocket streaming |
| ASR-03 | Recognition results display in real-time during recording | WebSocket send mechanism exists - needs display integration |
| ASR-04 | Final result is submitted when user releases hotkey | Existing hold_mode in global_hotkey.py |
| PUNC-01 | Punctuation model adds appropriate punctuation to recognized text | Punct-CT-Transformer integrated in server_init_recognizer.py |
| PUNC-02 | Bilingual punctuation support (Chinese and English) | Punct-CT-Transformer supports zh-en |
| CONF-01 | User can configure via config.py | config.py exists with ServerConfig and ClientConfig |
| CONF-02 | Hotkey trigger threshold is adjustable (default 0.3s) | Config.threshold = 0.3 in config.py |
| CONF-03 | Audio segment duration is configurable (default 25s) | Config.mic_seg_duration = 25 in config.py |
| SERV-01 | WebSocket server handles client connections | util/server/service.py with websockets library |
| SERV-02 | ASR runs in separate subprocess (non-blocking) | server_init_recognizer.py runs in subprocess via multiprocessing |
| SERV-03 | Server handles multiple audio segments with overlap | AudioCache and segmentation in server_ws_recv.py |
</phase_requirements>

## Standard Stack

### Core
| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| sherpa-onnx | latest | Offline ASR inference | Industry standard for local ONNX-based speech recognition |
| pynput | latest | Global hotkey detection | Cross-platform keyboard/mouse listener, already in use |
| websockets | latest | WebSocket server/client | Python async WebSocket standard library |
| asyncio | stdlib | Async event loop | Python async foundation |

### Supporting
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| numpy | latest | Audio data processing | Required for audio buffer handling |
| rich | latest | Terminal UI | Used in server for status display |
| base64 | stdlib | Audio encoding | For WebSocket message encoding |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| pynput | keyboard library | pynput already integrated, better cross-platform support |

**Installation:**
```bash
pip install sherpa-onnx pynput websockets numpy rich
```

## Architecture Patterns

### Recommended Project Structure
```
CapsWriter-Offline/
├── config.py                    # Configuration (exists)
├── core_client.py               # Client entry (exists)
├── core_server.py               # Server entry (exists)
├── util/
│   ├── server/
│   │   ├── service.py          # WebSocket server (exists)
│   │   ├── server_ws_recv.py  # Audio receive (exists)
│   │   ├── server_ws_send.py   # Result send (exists)
│   │   ├── server_recognize.py # ASR processing (exists)
│   │   ├── server_init_recognizer.py # Model init (exists)
│   │   └── text_merge.py       # Text merging (exists)
│   └── client/
│       ├── global_hotkey/      # Hotkey detection (exists)
│       ├── audio/recorder.py   # Audio capture (exists)
│       └── websocket_manager.py # WS client (exists)
```

### Pattern 1: Client/Server Architecture
**What:** Separate client (UI/input) and server (ASR subprocess) communicating via WebSocket
**When to use:** Phase 1 requirement SERV-01, SERV-02
**Example:**
```python
# Client: util/client/websocket_manager.py
class WebSocketManager:
    async def connect(self) -> bool:
        url = f"ws://{Config.addr}:{Config.port}"
        self.state.websocket = await websockets.connect(url)

# Server: util/server/service.py
async with websockets.serve(ws_handler, host, port):
    await asyncio.Future()
```
**Source:** Existing codebase - `util/client/websocket_manager.py`, `util/server/service.py`

### Pattern 2: Streaming Audio Segmentation
**What:** Continuous audio chunked into segments with overlap for streaming recognition
**When to use:** Phase 1 requirement ASR-02, ASR-03, SERV-03
**Example:**
```python
# From util/server/server_ws_recv.py
seg_threshold = seg_duration + seg_overlap * 2  # 25s + 2s*2 = 29s
while cache.duration >= seg_threshold:
    segment_data = cache.chunks[:segment_bytes]
    cache.chunks = cache.chunks[stride_bytes:]  # Slide by non-overlap
```
**Source:** Existing codebase - `util/server/server_ws_recv.py`

### Pattern 3: Dual Result Merging
**What:** Merge streaming results using both text overlap (robust) and token timestamp (precision)
**When to use:** Phase 1 requirement ASR-03, ASR-04
**Example:**
```python
# From util/server/server_recognize.py
_process_simple_merge(result, stream.result.text)  # text field
result.text_accu = tokens_to_text(result.tokens)   # timestamp-based
```
**Source:** Existing codebase - `util/server/text_merge.py`

### Anti-Patterns to Avoid
- **Blocking ASR in main thread:** ASR must run in subprocess to avoid blocking WebSocket heartbeat (SERV-02)
- **No overlap in streaming:** Without overlap, words at segment boundaries get cut (SERV-03)
- **Single result only:** Must handle both intermediate (real-time display) and final results (ASR-03, ASR-04)

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| ASR inference | Custom ONNX runtime | sherpa-onnx | Already integrated, supports multiple model types |
| Global hotkey | OS-level keyboard hooks | pynput | Already integrated, cross-platform |
| WebSocket streaming | Raw socket handling | websockets library | Already integrated, async native |
| Punctuation | Rule-based punctuation | Punct-CT-Transformer | Already integrated, bilingual support |
| Text deduplication | Simple concatenation | Token timestamp merge | Handles repeating words in streaming |

**Key insight:** The existing codebase already has all major components. The Phase 1 work is integration and ensuring all requirements are fully met.

## Common Pitfalls

### Pitfall 1: Audio Format Mismatch
**What goes wrong:** Recognition fails or produces garbage due to sample rate mismatch
**Why it happens:** Client sends 16kHz (downsampled from 48kHz) but server expects different format
**How to avoid:** Use consistent AudioFormat constants - client downsamples with `np.mean(data[::3], axis=1)` for 16kHz
**Warning signs:** Empty recognition results or UnicodeDecodeError in token processing

### Pitfall 2: Memory Leak from Result Cache
**What goes wrong:** `_results` dict in server_recognize.py grows unbounded
**Why it happens:** Results not cleaned up when client disconnects unexpectedly
**How to avoid:** Already implemented - `clear_results_by_socket_id()` called in finally block of ws_recv
**Warning signs:** Server memory usage grows over time

### Pitfall 3: Blocking Recognition in Server
**What goes wrong:** Long-running recognition blocks WebSocket server from accepting new connections
**Why it happens:** Single-threaded asyncio can't handle CPU-bound ASR
**How to avoid:** Already implemented - ASR runs in separate subprocess via multiprocessing Queue

### Pitfall 4: Hotkey Threshold Too Short
**What goes wrong:** Accidental noise triggers recording, or too long delays user input
**Why it happens:** 0.3s default works for most but may need tuning
**How to avoid:** Make configurable (CONF-02), default is already 0.3s in config.py

## Code Examples

### Hotkey Detection with pynput
```python
# Source: util/client/global_hotkey/global_hotkey.py
from pynput import keyboard

class GlobalHotkeyManager:
    def register(self, key_str: str, callback: Callable) -> None:
        self._hotkeys[key_str] = callback
        if self._running:
            self._restart_listener()

    def start(self) -> None:
        self._listener = keyboard.GlobalHotKeys(self._hotkeys)
        self._listener.start()
```

### Streaming Audio to Server
```python
# Source: util/client/audio/recorder.py
message = {
    'task_id': self.task_id,
    'seg_duration': Config.mic_seg_duration,  # 25s
    'seg_overlap': Config.mic_seg_overlap,    # 2s
    'is_final': False,
    'source': 'mic',
    'data': base64.b64encode(np.mean(data[::3], axis=1).tobytes()).decode(),
}
await websocket.send(json.dumps(message))
```

### ASR Recognition in Subprocess
```python
# Source: util/server/server_init_recognizer.py
def init_recognizer(queue_in: Queue, queue_out: Queue, sockets_id):
    # Load Sherpa-ONNX model
    recognizer = sherpa_onnx.OfflineRecognizer.from_funasr_nano(...)

    while True:
        task = queue_in.get()  # Blocking wait for audio
        result = recognize(recognizer, punc_model, task)
        queue_out.put(result)  # Send back to server
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| UDP for audio | WebSocket streaming | Pre-existing | Better reliability, connection state |
| Blocking ASR | Subprocess + Queue | Pre-existing | Non-blocking server |
| Single result | Dual merge (text + text_accu) | Pre-existing | Handles streaming artifacts |

**Deprecated/outdated:**
- None in scope for Phase 1

## Open Questions

1. **Real-time display implementation**
   - What we know: WebSocket sends results back, existing code has ResultProcessor
   - What's unclear: Exact UI mechanism (toast vs overlay vs terminal)
   - Recommendation: Use existing ResultProcessor pattern with configurable display

2. **Error handling specifics**
   - What we know: Error handlers exist in util/server/error_handler.py
   - What's unclear: Recovery strategies for different error types
   - Recommendation: Build on existing error_handler.py patterns

3. **Model selection defaults**
   - What we know: Config.model_type defaults to 'fun_asr_nano'
   - What's unclear: Whether to auto-detect best model for user's hardware
   - Recommendation: Keep current default, allow config change

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | pytest (not currently installed) |
| Config file | None — needs Wave 0 setup |
| Quick run command | `pytest tests/ -x` |
| Full suite command | `pytest tests/ --tb=short` |

### Phase Requirements → Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|--------------|
| ASR-01 | Hotkey triggers recording | unit | `pytest tests/test_hotkey.py -x` | ❌ Wave 0 |
| ASR-02 | Audio streams to server | integration | `pytest tests/test_streaming.py -x` | ❌ Wave 0 |
| ASR-03 | Real-time results display | unit | `pytest tests/test_display.py -x` | ❌ Wave 0 |
| ASR-04 | Release submits final | integration | `pytest tests/test_hotkey.py -x` | ❌ Wave 0 |
| PUNC-01 | Punctuation added | unit | `pytest tests/test_punctuation.py -x` | ❌ Wave 0 |
| PUNC-02 | Bilingual support | unit | `pytest tests/test_punctuation.py -x` | ❌ Wave 0 |
| CONF-01 | Config via config.py | unit | `pytest tests/test_config.py -x` | ❌ Wave 0 |
| CONF-02 | Threshold adjustable | unit | `pytest tests/test_config.py -x` | ❌ Wave 0 |
| CONF-03 | Segment duration config | unit | `pytest tests/test_config.py -x` | ❌ Wave 0 |
| SERV-01 | WebSocket server | integration | `pytest tests/test_server.py -x` | ❌ Wave 0 |
| SERV-02 | ASR subprocess | integration | `pytest tests/test_server.py -x` | ❌ Wave 0 |
| SERV-03 | Multiple segments | integration | `pytest tests/test_streaming.py -x` | ❌ Wave 0 |

### Sampling Rate
- **Per task commit:** `pytest tests/ -x` (fails fast on first error)
- **Per wave merge:** `pytest tests/ --tb=short` (full suite)
- **Phase gate:** Full suite green before `/gsd:verify-work`

### Wave 0 Gaps
- [ ] `tests/test_hotkey.py` — covers ASR-01, ASR-04
- [ ] `tests/test_streaming.py` — covers ASR-02, ASR-03, SERV-03
- [ ] `tests/test_punctuation.py` — covers PUNC-01, PUNC-02
- [ ] `tests/test_config.py` — covers CONF-01, CONF-02, CONF-03
- [ ] `tests/test_server.py` — covers SERV-01, SERV-02
- [ ] `tests/conftest.py` — shared fixtures
- [ ] Framework install: `pip install pytest` — add to requirements

## Sources

### Primary (HIGH confidence)
- Existing codebase in `util/server/` and `util/client/` — sherpa-onnx, pynput, websockets integration
- sherpa-onnx official docs: https://k2-fsa.github.io/sherpa/onnx/ — API for OfflineRecognizer

### Secondary (MEDIUM confidence)
- pynput docs: https://pynput.readthedocs.io/ — global hotkey implementation

### Tertiary (LOW confidence)
- None required — existing code is well-documented and functional

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH - All libraries already integrated in existing codebase
- Architecture: HIGH - C/S architecture already established
- Pitfalls: HIGH - All identified pitfalls already handled in existing code

**Research date:** 2026-03-08
**Valid until:** 2026-09-08 (stable codebase, no fast-moving changes expected)
