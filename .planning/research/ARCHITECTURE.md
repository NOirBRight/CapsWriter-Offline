# Architecture Patterns

**Project:** CapsWriter-Offline
**Domain:** Offline Voice Dictation Software
**Researched:** 2026-03-08
**Confidence:** HIGH (based on existing codebase analysis + project documentation)

## Executive Summary

CapsWriter-Offline uses a **Client-Server (C/S) architecture** with separate processes for UI/input handling and CPU-intensive AI inference. This separation ensures that heavy ASR model execution does not block the WebSocket heartbeat or UI responsiveness. The architecture follows a streaming pipeline pattern with real-time audio chunking, WebSocket-based communication, and multi-stage post-processing.

## Recommended Architecture

### High-Level Diagram

```
+------------------+     WebSocket      +------------------+
|     CLIENT       | <================> |      SERVER      |
|   (Main App)    |                    |  (ASR Engine)   |
+------------------+                    +------------------+
        |                                        |
        v                                        v
+------------------+                    +------------------+
|  - Hotkey (Cap  |                    |  - Sherpa-ONNX  |
|  - Audio Record |                    |  - FunASR/Paraf |
|  - UI (Tkinter) |                    |  - Punct Model  |
|  - Text Output  |                    +------------------+
+------------------+
        |
        v
+------------------+
|   LLM Polishing  |  (Optional, async)
|   - Ollama       |
|   - Hotword RAG  |
+------------------+
```

### Component Boundaries

| Component | Responsibility | Communicates With |
|-----------|---------------|-------------------|
| **Client (Main Process)** | Global hotkey detection, audio capture, UI display, keyboard simulation | Server (WebSocket), LLM (HTTP), Hotword RAG (direct) |
| **Server (Subprocess)** | ASR model loading/inference, punctuation, text merging | Client (WebSocket), ONNX Runtime |
| **Audio Capture** | Microphone streaming, chunking, segment duration control | Client |
| **WebSocket Handler** | Real-time streaming, connection management | Client <-> Server |
| **ASR Engine** | Offline speech recognition (Sherpa-ONNX) | Server |
| **Punctuation Model** | Offline punctuation restoration | Server |
| **Hotword RAG** | Phoneme-based fuzzy matching | Client, hot.txt files |
| **LLM Agent** | Text polishing, role-based responses | Ollama/local LLM providers |
| **Text Output** | Keyboard simulation, clipboard, toast display | Client, OS input system |
| **History/Diary** | Audio/transcript archival | Client, filesystem |

### Data Flow

#### Primary Flow: Voice Dictation

```
1. USER PRESS CAPSLOCK
       |
       v
2. CLIENT: Start audio capture (sounddevice stream)
       |
       v
3. CLIENT: Collect audio chunks (25s segments, 2s overlap)
       |
       v
4. CLIENT: Send chunk via WebSocket to SERVER
       |
       v
5. SERVER: Receive chunk, pass to Sherpa-ONNX
       |
       v
6. SERVER: ASR inference returns partial results
       |         - text (fuzzy dedup)
       |         - text_accu (token timestamp dedup)
       v
7. SERVER: Send partial results back via WebSocket
       |
       v
8. CLIENT: Receive partial, update UI (optional)
       |
       v
9. USER RELEASE CAPSLOCK
       |
       v
10. CLIENT: Send "final" signal to SERVER
       |
       v
11. SERVER: Return final results with punctuation
       |
       v
12. CLIENT: Post-process
       |  - Hotword RAG (phoneme matching)
       |  - LLM polishing (optional)
       |  - Rule replacement
       v
13. CLIENT: Output text
       |  - keyboard.write() for direct typing
       |  - Toast display for review
       |  - Clipboard fallback
       v
14. COMPLETE
```

#### Secondary Flow: Audio File Transcription

```
1. USER DRAG AUDIO FILE
       |
       v
2. CLIENT: ffmpeg extract audio
       |
       v
3. CLIENT: Send full audio to SERVER
       |
       v
4. SERVER: Process with ASR + get timestamps
       |
       v
5. SERVER: Return timestamped results
       |
       v
6. CLIENT: Generate .srt file
       |
       v
7. CLIENT: Save to history archive
       |
       v
8. COMPLETE
```

## Patterns to Follow

### Pattern 1: Process Separation for CPU-Bound Work

**What:** Run ASR inference in a separate subprocess from the UI/main application.

**When:** The ASR model (Sherpa-ONNX with FunASR/Paraformer) consumes significant CPU, which would block WebSocket heartbeats and make the UI unresponsive.

**Why:** This is already implemented correctly in CapsWriter-Offline. The Server subprocess handles all AI inference while the Client process manages UI and I/O.

**Example:**
```python
# Server process (subprocess)
import sherpa_onnx
recognizer = sherpa_onnx.OnlineRecognizer(...)
while True:
    audio_chunk = await websocket.recv()
    result = recognizer.accept_waveform(audio_chunk)
    await websocket.send(result)
```

### Pattern 2: Dual Result Text Merging

**What:** Compute two text outputs: `text` (simple concatenation with fuzzy dedup) and `text_accu` (token timestamp-based deduplication).

**When:** Streaming ASR returns overlapping results; simple concatenation causes duplicates.

**Why:** Different dedup strategies suit different speech patterns. `text` is more robust, `text_accu` is more precise.

**Example:**
```python
# From util/server/text_merge.py
def merge_text(results):
    # text: fuzzy matching dedup
    # text_accu: token timestamp dedup
    return {'text': text, 'text_accu': text_accu}
```

### Pattern 3: Chunked Streaming with Overlap

**What:** Send audio in segments (e.g., 25 seconds) with overlap (e.g., 2 seconds) to preserve context at segment boundaries.

**When:** Long dictation sessions that exceed single inference capacity.

**Why:** Prevents context loss at chunk boundaries; overlapping audio ensures smooth transitions.

**Example:**
```python
# Client config
mic_seg_duration = 25  # seconds
mic_seg_overlap = 2   # seconds
```

### Pattern 4: Two-Stage Hotword Retrieval

**What:** FastRAG (Numba JIT inverted index) for coarse filtering, then AccuRAG (phoneme fuzzy matching) for precise results.

**When:** Large hotword lists that would be slow to search linearly.

**Why:** Reduces computation by 90% through initial coarse filtering before expensive fuzzy matching.

### Pattern 5: Hot-Reload Configuration

**What:** Use `watchdog` to monitor config files (`hot.txt`, `LLM/*.py`) and reload on change without restart.

**When:** Users frequently update hotwords or LLM roles.

**Why:** Improves UX; no restart needed for configuration changes.

## Anti-Patterns to Avoid

### Anti-Pattern 1: Blocking the Main Thread with ASR

**What:** Running Sherpa-ONNX inference in the main Client process.

**Why bad:** ASR inference is CPU-intensive; blocking the main thread freezes UI, prevents WebSocket heartbeat, causes timeouts.

**Instead:** Keep C/S architecture with Server subprocess for inference.

### Anti-Pattern 2: VAD-Based Slicing for Dictation

**What:** Using Voice Activity Detection to determine when to start/stop sending audio.

**Why bad:** VAD cuts off audio at natural pauses, losing context for the next chunk. For dictation, continuous context matters more than VAD efficiency.

**Instead:** Time-based slicing with fixed duration (e.g., 25s segments) preserves context across boundaries.

### Anti-Pattern 3: Direct Text Replacement for Hotwords

**What:** Simple string replacement of hotwords in ASR output.

**Why bad:** Chinese homophones (shi/xi, de/di) cannot be distinguished by text alone. Wrong character chosen.

**Instead:** Phoneme-based RAG retrieves alternatives based on pronunciation similarity.

### Anti-Pattern 4: Synchronous LLM Calls Blocking Dictation Flow

**What:** Calling LLM synchronously after dictation completes, blocking text output.

**Why bad:** LLM calls can take seconds; user waits for text to appear.

**Instead:** LLM polishing is optional and async; dictation text appears immediately, LLM result comes later or is merged.

## Scalability Considerations

| Concern | At 100 users | At 10K users | At 1M users |
|---------|--------------|--------------|-------------|
| **Latency** | <500ms typical | <1s typical | Requires optimization |
| **Model loading** | Single model per server | Model caching | Distributed servers |
| **Memory** | ~2GB RAM (ASR + app) | Same | Per-instance |
| **Concurrent dictations** | Single user | Single user | Single user (local app) |
| **Hotword lookup** | Linear scan OK | Needs FastRAG | Needs FastRAG |
| **History storage** | Local filesystem | Local filesystem | Consider SQLite |

**Note:** This is a local desktop application, not a multi-user server. Scalability concerns are primarily about the single-user experience, not concurrent users.

## Build Order Implications

### Phase Dependencies

```
Phase 1: Core Infrastructure
    ├── Client basic skeleton (hotkey + audio capture)
    ├── Server WebSocket echo
    └── Basic connectivity test
           |
           v
Phase 2: ASR Integration
    ├── Sherpa-ONNX integration in Server
    ├── Streaming audio pipeline
    ├── Text result handling
    └── Punctuation model
           |
           v
Phase 3: Output & Polish
    ├── Keyboard simulation
    ├── Hotword system (FastRAG + AccuRAG)
    ├── Toast display UI
    └── Basic LLM integration
           |
           v
Phase 4: Features & Polish
    ├── Role system
    ├── Audio file transcription
    ├── History/diary archive
    └── Multi-model support
```

### Why This Order

1. **Infrastructure first:** Establish the C/S communication before adding any AI. If WebSocket fails, nothing else works.

2. **ASR integration second:** This is the core value proposition. Get working dictation before adding polish.

3. **Output & polish third:** Once dictation works, add what makes it usable—hotwords fix errors, LLM improves quality, keyboard simulation delivers output.

4. **Features last:** Transcription, history, and multi-model are nice-to-have enhancements built on the working core.

### Critical Path Items

| Item | Blocks | Why Critical |
|------|--------|---------------|
| WebSocket connectivity | All subsequent phases | No communication = no dictation |
| Streaming audio pipeline | Dictation flow | Core user experience |
| Sherpa-ONNX inference | ASR phase | Core value proposition |
| Keyboard simulation | Output delivery | Dictation must produce text |
| Hotword RAG | Error correction | Differentiator from competitors |

## Sources

- Project CLAUDE.md (existing architecture documentation)
- Project PROJECT.md (requirements and context)
- Sherpa-ONNX documentation (https://k2-fsa.github.io/sherpa/onnx/)
- FunASR GitHub (https://github.com/modelscope/FunASR)
- Project codebase: `util/server/text_merge.py`, `util/client/`, `util/hotword/`
