# Codebase Structure

**Analysis Date:** 2026-03-08

## Directory Layout

```
CapsWriter-Offline/
├── core_client.py           # Client entry point
├── core_server.py          # Server entry point
├── config.py               # Configuration definitions
├── config.json             # GUI configuration (user-editable)
├── gui/                    # GUI application (Tkinter)
│   ├── main_window.py      # Main window
│   ├── config_manager.py   # Config loading/saving
│   └── panels/             # Config panels (ASR, Shortcut, LLM, etc.)
├── util/                   # Shared utilities
│   ├── client/             # Client-side modules
│   ├── server/             # Server-side modules
│   ├── llm/                # LLM integration
│   ├── hotword/            # Hotword matching
│   ├── fun_asr_gguf/       # Fun-ASR GGUF model
│   ├── logger.py           # Logging setup
│   ├── protocol.py         # Communication protocol
│   └── ...
├── models/                 # AI models
│   ├── Fun-ASR-Nano/      # ASR model
│   ├── Paraformer/        # ASR model
│   └── Punct-CT-Transformer/ # Punctuation model
├── LLM/                    # LLM role definitions
├── logs/                   # Log files
├── build.spec              # PyInstaller spec (server+client)
├── build-client.spec       # PyInstaller spec (client only)
└── CapsWriter-GUI.exe      # Built executable
```

## Directory Purposes

**Root Directory:**
- Purpose: Entry points and main configuration
- Contains: `core_client.py`, `core_server.py`, `config.py`, `config.json`
- Key files: `config.json` (GUI settings), `hot.txt` (hotwords)

**`gui/` Directory:**
- Purpose: Tkinter-based configuration GUI
- Contains: Main window, config panels
- Key files: `gui/main_window.py`, `gui/config_manager.py`

**`util/` Directory:**
- Purpose: Core functionality modules
- Contains: Client, server, LLM, hotword, and supporting utilities
- Structure: Organized by concern (client/server/llm/hotword)

**`util/client/` Submodules:**
- `audio/recorder.py` - Microphone recording
- `clipboard/clipboard.py` - Clipboard operations
- `global_hotkey/global_hotkey.py` - Keyboard shortcuts
- `output/text_output.py` - Text output (typing simulation)
- `transcribe/` - File transcription
- `websocket_manager.py` - WebSocket client

**`util/server/` Submodules:**
- `server_ws_recv.py` - WebSocket receive (audio ingestion)
- `server_ws_send.py` - WebSocket send (result output)
- `server_recognize.py` - ASR recognition
- `text_merge.py` - Text segment merging
- `server_classes.py` - Task/Result dataclasses

**`util/hotword/` Submodules:**
- `manager.py` - Hotword management
- `rag_fast.py` - Fast RAG retrieval
- `rag_accu.py` - Accurate RAG retrieval
- `hot_phoneme.py` - Phoneme conversion
- `hot_rule.py` - Rule-based replacement

**`util/llm/` Submodules:**
- `llm_client_pool.py` - LLM provider management
- `llm_context.py` - Context assembly
- `llm_clipboard.py` - Clipboard-based context

**`models/` Directory:**
- Purpose: AI model files (ONNX format)
- Contains: ASR models, punctuation models
- Generated: No (downloaded separately)

**`LLM/` Directory:**
- Purpose: LLM role definitions
- Contains: Python files defining roles (e.g., `default.py`, `翻译.py`)
- Key files: `LLM/default.py` - Default role template

**`logs/` Directory:**
- Purpose: Runtime log files
- Contains: `client.log`, `server.log`
- Generated: Yes (runtime)
- Committed: No

## Key File Locations

**Entry Points:**
- `core_client.py`: Client application entry
- `core_server.py`: Server application entry
- `gui_launcher.py`: GUI application entry

**Configuration:**
- `config.py`: Configuration class definitions (ServerConfig, ClientConfig)
- `config.json`: GUI configuration file (runtime, user-editable)
- `config.json.example`: Configuration template

**Core Logic:**
- `util/client/state.py`: Client state management
- `util/server/server_recognize.py`: ASR recognition logic
- `util/server/text_merge.py`: Text merging algorithms
- `util/hotword/manager.py`: Hotword system

**Testing:**
- Test files co-located with model directories (e.g., `models/*/test.py`)

## Naming Conventions

**Files:**
- Python modules: `snake_case.py` (e.g., `core_client.py`, `server_ws_recv.py`)
- GUI panels: `snake_case.py` (e.g., `asr_panel.py`, `shortcut_panel.py`)
- LLM roles: Chinese or English descriptive (e.g., `翻译.py`, `default.py`)

**Directories:**
- Modules: `snake_case` (e.g., `util/client`, `util/server`)
- Models: Descriptive (e.g., `Fun-ASR-Nano`, `Paraformer`)

**Classes:**
- PascalCase: `ClientState`, `ServerConfig`, `MainWindow`

**Functions/Variables:**
- snake_case: `get_state()`, `run_websocket_server()`

## Where to Add New Code

**New Feature (Client):**
- Implementation: `util/client/<feature>/`
- Tests: Model test directory

**New Feature (Server):**
- Implementation: `util/server/`
- Tests: Model test directory

**New Component/Module:**
- Implementation: `util/<module>/`
- Configuration: Update `config.py` if needed

**New LLM Role:**
- Implementation: `LLM/<role_name>.py`
- Follow template in `LLM/__init__.py`

**New Hotword:**
- Add to: `hot.txt` (phoneme-based) or `hot-rule.txt` (regex)

**Utilities:**
- Shared helpers: `util/` (appropriate submodule)
- Logging: Use `util/logger.py`

## Special Directories

**`models/`:**
- Purpose: AI model files
- Generated: No (downloaded from external sources)
- Committed: Yes (in git, large binary files)

**`logs/`:**
- Purpose: Runtime logs
- Generated: Yes (runtime)
- Committed: No (in .gitignore)

**`build/`:**
- Purpose: Build artifacts
- Generated: Yes (during build)
- Committed: No

**`dist/`:**
- Purpose: Distribution packages
- Generated: Yes
- Committed: No

**`_internal/`:**
- Purpose: PyInstaller internal files
- Generated: Yes (packaging)
- Committed: No

---

*Structure analysis: 2026-03-08*
