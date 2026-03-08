# Technology Stack

**Analysis Date:** 2026-03-08

## Languages

**Primary:**
- Python 3.x - Core application logic, ASR processing, LLM integration

**Secondary:**
- None detected (pure Python application)

## Runtime

**Environment:**
- Python 3.x (standard runtime)
- PyInstaller 6.0+ for Windows executable packaging

**Package Manager:**
- pip (requirements.txt based)
- Lockfile: Not present (direct requirements files used)

## Frameworks

**Core:**
- Sherpa-ONNX - Offline automatic speech recognition (ASR) engine
- ONNX Runtime - CPU/GPU model inference
- websockets - Client-server real-time communication

**GUI:**
- Tkinter - Desktop GUI framework (built-in Python)
- ttkbootstrap - Modern Tkinter theming
- pystray - System tray integration
- Pillow (PIL) - Image processing for icons

**Input/Device:**
- pynput - Global keyboard/mouse event monitoring
- keyboard - Global hotkey registration
- pyclip - System clipboard access
- sounddevice - Audio recording from microphone

**Data Processing:**
- numpy - Numerical computing
- numba - JIT compilation for performance-critical code
- pypinyin - Chinese pinyin conversion for hotword matching

**LLM Integration:**
- openai - OpenAI-compatible API client (used for Ollama and OpenAI)

**Build:**
- PyInstaller - Windows executable packaging

**Utilities:**
- watchdog - File system monitoring for hot-reload
- rich - Terminal formatting
- markdown - Markdown rendering
- srt - SRT subtitle file handling

## Key Dependencies

**ASR/Speech:**
- sherpa-onnx 1.12.20+cuda12.cudnn9 - Offline speech recognition
- onnxruntime - Model inference engine

**Audio:**
- sounddevice - Audio capture from microphone

**GUI:**
- tkinter - Desktop GUI
- ttkbootstrap - GUI theming
- pystray - System tray icon
- Pillow - Image handling
- tkhtmlview - HTML rendering in Tkinter

**Input:**
- keyboard - Global hotkey registration
- pynput - Keyboard/mouse event listening
- pyclip - Clipboard read/write

**LLM:**
- openai - OpenAI-compatible API client
- httpx - HTTP client for API calls

**Performance:**
- numpy - Array operations
- numba - JIT acceleration

## Configuration

**Environment:**
- JSON-based configuration: `config.json`
- Python config module: `config.py` with ServerConfig class

**Key settings:**
- ASR model selection (fun_asr_nano, sensevoice, paraformer)
- Vulkan GPU acceleration options
- WebSocket server port (6016)
- LLM provider configurations

**Build:**
- PyInstaller spec files:
  - `build.spec` - Full server+client build
  - `build-client.spec` - Client-only build
  - `build-gui.spec` - GUI launcher build

## Platform Requirements

**Development:**
- Python 3.x
- Windows OS (primary target)
- Microphone audio input
- GPU optional (Vulkan support for acceleration)

**Production:**
- Windows executable (.exe) via PyInstaller
- Optional: GPU with Vulkan support for faster inference
- Models loaded from local `models/` directory

**Runtime Requirements:**
- WebSocket server running on configurable port
- Ollama (optional) for local LLM on localhost:11434
- Microphone access for voice input

---

*Stack analysis: 2026-03-08*
