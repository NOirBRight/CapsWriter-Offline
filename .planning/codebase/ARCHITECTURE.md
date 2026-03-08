# Architecture

**Analysis Date:** 2026-03-08

## Pattern Overview

**Overall:** Client-Server Architecture with Process Isolation

**Key Characteristics:**
- **C/S Architecture**: Server handles AI inference in a separate process; Client handles UI, hotkey listening, and audio capture
- **WebSocket Communication**: Binary protocol between client and server for real-time streaming
- **Offline-First**: All AI models run locally (ASR, punctuation, LLM) for privacy
- **Configuration-Driven**: `config.py` and JSON config support hot-reload

## Layers

**Client Layer:**
- Purpose: User interface, audio capture, hotkey listening, result output
- Location: `core_client.py` (entry), `util/client/` (modules)
- Contains: GUI (Tkinter), audio recording, keyboard/mouse simulation, WebSocket client
- Depends on: `config.py`, `util/server/`, `util/hotword/`, `util/llm/`
- Used by: End users

**Server Layer:**
- Purpose: Audio recognition, text processing, model inference
- Location: `core_server.py` (entry), `util/server/` (modules)
- Contains: WebSocket server, ASR recognizer, text merging, punctuation model
- Depends on: `config.py`, models in `models/`, `util/hotword/`
- Used by: Client via WebSocket

**GUI Layer:**
- Purpose: Configuration UI for users to modify settings
- Location: `gui/` (Tkinter-based)
- Contains: Main window, config panels for ASR/LLM/shortcuts
- Depends on: `config.py`, `config.json`
- Used by: End users

**Utility Layer:**
- Purpose: Shared functionality across client/server
- Location: `util/`
- Submodules:
  - `util/client/` - Client utilities (audio, clipboard, hotkey, output)
  - `util/server/` - Server utilities (WebSocket, recognition, text merge)
  - `util/llm/` - LLM client pool, context management
  - `util/hotword/` - Phoneme-based RAG hotword matching
  - `util/fun_asr_gguf/` - Fun-ASR GGUF model integration

## Data Flow

**Voice Recognition Flow:**

1. **Trigger**: User presses hotkey (e.g., CapsLock)
2. **Capture**: Client records audio chunks via `util/client/audio/recorder.py`
3. **Stream**: Client streams audio via WebSocket to Server (`util/server/server_ws_recv.py`)
4. **Segment**: Server buffers audio and segments by time threshold
5. **Recognize**: Server runs ASR model (`util/server/server_recognize.py`)
6. **Merge**: Server merges text segments using timestamp or fuzzy matching
7. **Return**: Server sends result via WebSocket (`util/server/server_ws_send.py`)
8. **Post-process**: Client applies hotword replacement (`util/hotword/`)
9. **LLM Polish**: Optional LLM polish if enabled (`util/llm/`)
10. **Output**: Client types result or shows in Toast

**Client-Server Communication:**
- Protocol: WebSocket with binary subprotocol
- Message format: JSON with base64-encoded audio data
- Port: 6016 (configurable)

## Key Abstractions

**ClientState:**
- Purpose: Manages global client state including WebSocket, audio stream, recording status
- Examples: `util/client/state.py`
- Pattern: Dataclass with singleton pattern via `get_state()`

**Task/Result:**
- Purpose: Data structures for recognition tasks and results
- Examples: `util/server/server_classes.py`
- Pattern: Dataclass

**Cosmic (Server State):**
- Purpose: Server-wide shared state (sockets, queues, recognizer)
- Examples: `util/server/server_cosmic.py`
- Pattern: Global singleton

**Hotword Manager:**
- Purpose: Phoneme-based RAG matching for hotword replacement
- Examples: `util/hotword/manager.py`, `util/hotword/rag_fast.py`
- Pattern: Two-stage retrieval (FastRAG + AccuRAG)

**LLM Client Pool:**
- Purpose: Manages multiple LLM providers (Ollama, OpenAI, DeepSeek, etc.)
- Examples: `util/llm/llm_client_pool.py`
- Pattern: Provider abstraction with fallback

## Entry Points

**Client Entry:**
- Location: `core_client.py`
- Triggers: `python core_client.py` or GUI launcher
- Responsibilities: Initialize lifecycle, setup components, run main loop (mic or file mode)

**Server Entry:**
- Location: `core_server.py`
- Triggers: `python core_server.py` or start via GUI
- Responsibilities: Initialize Vulkan/GPU config, start recognizer process, run WebSocket server

**GUI Entry:**
- Location: `gui_launcher.py` / `CapsWriter-GUI.pyw`
- Triggers: Launch executable or script
- Responsibilities: Create main window, manage server/client processes, save config

**File Transcription Mode:**
- Location: `core_client.py` with file arguments
- Triggers: `python core_client.py file.mp4`
- Responsibilities: Transcode file via ffmpeg, send to server, receive with timestamps, generate SRT

## Error Handling

**Strategy:** Lifecycle-based graceful shutdown with resource cleanup

**Patterns:**
- `util/common/lifecycle.py`: Manages startup/shutdown signals, registered cleanup callbacks
- Try-except blocks in main loops with logging
- WebSocket connection error handling with reconnection logic
- Model loading errors with fallback to alternatives

## Cross-Cutting Concerns

**Logging:** `util/logger.py` - Centralized logging setup with configurable levels

**Configuration:** `config.py` - ServerConfig/ClientConfig classes; `config.json` for GUI

**Hotwords:** `hot.txt`, `hot-rule.txt`, `hot-rectify.txt` - Text-based hotword files with hot-reload

**LLM Roles:** `LLM/*.py` - Modular role definitions with prompt templates

---

*Architecture analysis: 2026-03-08*
