# External Integrations

**Analysis Date:** 2026-03-08

## APIs & External Services

**Local LLM (Ollama):**
- Service: Ollama - Local LLM runtime
- Purpose: Text polishing, translation, and AI-powered corrections
- Base URL: `http://localhost:11434/v1`
- API Key: Not required (local)
- Configured models: `minicpm-v:latest`, `qwen2.5:7b`
- Files: `LLM/*.py` (role definitions)

**OpenAI API (Optional):**
- Service: OpenAI GPT series
- Purpose: Alternative LLM provider when Ollama unavailable
- SDK: `openai` Python package
- Config: Via `config.json` with custom base_url and api_key
- Implementation: `util/llm/llm_client_pool.py`

**Local Whisper API (Optional):**
- Service: Local Whisper-compatible API
- Purpose: Alternative speech recognition
- Config: Via config.json alternative providers

## Data Storage

**Configuration:**
- Format: JSON
- File: `config.json`
- Location: Project root

**Hotword Data:**
- Files:
  - `hot.txt` - Unified Chinese/English hotwords (phoneme-based RAG)
  - `hot-rule.txt` - Regex rule replacements
  - `hot-rectify.txt` - Historical correction RAG
- Location: Project root
- Format: Plain text (line-separated)

**LLM Roles:**
- Directory: `LLM/`
- Format: Python modules defining Role classes
- Examples: `default.py`, `翻译.py`, `高级翻译.py`, `Python.py`, `命令.py`, `大助理.py`

**Diary/History:**
- Format: Markdown with HTML audio embed
- Location: `年份/月份/日期.md`
- Audio: `年份/月份/assets/`

**Model Files:**
- Location: `models/` directory
- Formats: ONNX, GGUF
- Models: Fun-ASR-Nano, SenseVoice, Paraformer, Punct-CT-Transformer

## Authentication & Identity

**No external authentication required:**
- Application is fully offline-capable
- LLM uses local Ollama (no auth) or custom API keys

## Speech Recognition Models (Offline)

**Sherpa-ONNX Models:**
- Fun-ASR-Nano: Default model, fastest
- SenseVoice: Multi-language support (zh/en/ja/ko/yue)
- Paraformer: High accuracy, larger model
- Punct-CT-Transformer: Punctuation restoration

**Model Loading:**
- Implementation: `util/server/server_init_recognizer.py`
- Configuration: Via `config.py` ServerConfig.model_type

## Monitoring & Observability

**Logging:**
- Files: `logs/client.log`, `logs/server.log`
- Implementation: Python logging module
- Configurable via `config.py` ServerConfig.log_level

**No external monitoring services:**
- Application is designed for offline operation

## CI/CD & Deployment

**Build System:**
- PyInstaller - Windows executable packaging
- Spec files: `build.spec`, `build-client.spec`, `build-gui.spec`

**Distribution:**
- Windows executable: `CapsWriter-GUI.exe`
- Self-contained with bundled Python runtime
- Models and config loaded from project directory

## Communication Protocols

**Client-Server Communication:**
- Protocol: WebSocket
- Port: 6016 (configurable via config.py)
- Implementation:
  - Server: `util/server/server_ws_recv.py`, `core_server.py`
  - Client: `util/client/websocket_manager.py`
- Purpose: Real-time streaming of audio chunks and recognition results

**UDP Control:**
- Implementation: `util/client/udp/udp_control.py`
- Purpose: Control signals between client and server

## Environment Configuration

**Required configurations in config.json:**
- ASR model type
- Vulkan GPU settings
- Scene definitions (hotkeys, processors)
- LLM provider configs (base_url, api_key, model)
- Overlay display settings

**Runtime dependencies:**
- Ollama service (optional, for LLM features)
- Microphone access
- GPU (optional, for acceleration)

## Webhooks & Callbacks

**No webhooks:**
- Fully offline operation
- All processing done locally

**Internal callbacks:**
- Keyboard event handlers via pynput
- WebSocket message callbacks
- File watchdog callbacks for hot-reload

---

*Integration audit: 2026-03-08*
