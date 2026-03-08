# Technology Stack

**Project:** CapsWriter-Offline
**Researched:** 2026-03-08
**Confidence:** MEDIUM (based on existing codebase analysis + training data; web search unavailable for verification)

## Recommended Stack

### Core Framework
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| Python | 3.10-3.12 | Application runtime | Standard for ML/audio; 3.10+ has best package compatibility |
| PyInstaller | 6.0+ | Executable packaging | Industry standard for Python Windows builds; supports CUDA collect |

### ASR Engine
| Technology | Purpose | Why |
|------------|---------|-----|
| **Sherpa-ONNX** | Offline ASR | Best-in-class for offline; ONNX runtime is fast; supports multiple models |
| **FunASR-Nano** | Chinese ASR model | Optimized for Chinese; GGUF format for CPU efficiency |
| **Paraformer** | High-accuracy ASR | Larger model for better accuracy when resources allow |
| **SenseVoice** | Multi-language | Supports zh/en/ja/ko/yue for multilingual users |
| **Punct-CT-Transformer** | Punctuation | Offline punctuation restoration |

**Why Sherpa-ONNX:** Native ONNX support, no PyTorch dependency at runtime, streaming support, active development. The project already uses this stack effectively.

### Audio Processing
| Library | Version | Purpose | Why |
|---------|---------|---------|-----|
| sounddevice | Latest | Mic capture | Cross-platform audio; integrates well with numpy |
| numpy | 1.24+ | Audio arrays | Essential for audio processing; efficient |
| numba | Latest | JIT acceleration | 10x speedup for RAG hotword matching |

**Why sounddevice:** Already in use; better than pyaudio (no binary dependencies). Works well with the async architecture.

### Input/Device Control
| Library | Purpose | Why |
|---------|---------|-----|
| **pynput** | Global hotkeys, keyboard simulation | Already used; handles CapsLock monitoring well |
| **keyboard** | Text typing simulation | Already used; `keyboard.write()` avoids IME conflicts |
| **pyclip** | Clipboard access | Cross-platform; already integrated |

**Why pynput + keyboard combo:** pynput for global hotkey detection, keyboard for smooth text output. Avoids Chinese IME interference issues.

### WebSocket Communication
| Library | Purpose | Why |
|---------|---------|-----|
| websockets | Real-time streaming | Async-native; already used; handles reconnection |

### LLM Integration
| Library | Purpose | Why |
|---------|---------|-----|
| openai | Ollama/OpenAI API | Already used; supports any OpenAI-compatible endpoint |
| httpx | HTTP client | Async HTTP; used for API calls |

### GUI
| Library | Purpose | Why |
|---------|---------|-----|
| tkinter | Desktop UI | Built-in; already used |
| ttkbootstrap | Modern theming | Already used; clean look |
| pystray | System tray | Already used |
| Pillow | Image processing | Already used for icons |
| tkhtmlview | Markdown rendering | Already used for LLM output |

### Infrastructure
| Technology | Purpose | Why |
|------------|---------|-----|
| onnxruntime | Model inference | Already used; CPU/GPU support |
| gguf | GGUF model format | Already used; efficient CPU loading |

### Supporting Libraries
| Library | Purpose | When to Use |
|---------|---------|--------------|
| rich | Terminal output | Status displays, progress bars |
| watchdog | File monitoring | Hot-reload of config/hotwords |
| pypinyin | Chinese pinyin | Hotword phoneme matching |
| srt | Subtitle handling | Audio file transcription to SRT |
| markdown | MD rendering | LLM output display |

## Alternatives Considered

| Category | Recommended | Alternative | Why Not |
|----------|-------------|-------------|---------|
| ASR | Sherpa-ONNX | faster-whisper | Whisper is larger; Sherpa-ONNX has better Chinese support |
| Audio | sounddevice | pyaudio | sounddevice has fewer Windows binary issues |
| Hotkeys | pynput | keyboard (package) | pynput handles modifier-only keys (CapsLock) better |
| GUI | tkinter | PyQt/PySide | tkinter is built-in; PyQt is heavier |
| Packaging | PyInstaller | cx_Freeze/Nuitka | PyInstaller has best Windows .exe support |

## Installation

```bash
# Core runtime (conda recommended)
conda create -n capswriter python=3.11
conda activate capswriter

# Client dependencies
pip install -r requirements-client.txt

# Server dependencies (includes Sherpa-ONNX)
pip install -r requirements-server.txt

# Build
pyinstaller build.spec
```

## Current Project Dependencies

**requirements-client.txt:**
- rich, typer, colorama, markdown
- keyboard, pynput, pyclip, sounddevice, watchdog
- websockets, openai, httpx
- numpy, numba, pypinyin, srt
- pystray, Pillow, tkhtmlview
- pyinstaller

**requirements-server.txt:**
- sherpa-onnx (with CUDA), numpy, gguf, onnxruntime
- rich, websockets, watchdog
- pystray, Pillow, markdown, tkhtmlview

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| ASR Stack | HIGH | Sherpa-ONNX is established; project uses it successfully |
| Audio | HIGH | sounddevice + numpy proven in production |
| Input | HIGH | pynput + keyboard combo works well |
| WebSocket | HIGH | websockets library is standard |
| GUI | HIGH | tkinter + ttkbootstrap proven |
| Packaging | HIGH | PyInstaller 6.0+ documented in project |

## Gaps to Address

- **Version pinning:** requirements*.txt should pin exact versions for reproducibility
- **PyTorch dependency:** Some model export scripts use torch; consider if runtime needs it
- **CUDA optionality:** Server requires CUDA build; CPU-only fallback needs testing

## Sources

- Project codebase analysis (existing implementation)
- Sherpa-ONNX documentation (https://k2-fsa.github.io/sherpa/onnx/)
- Project CLAUDE.md (architectural decisions)
- requirements-client.txt / requirements-server.txt (current dependencies)
