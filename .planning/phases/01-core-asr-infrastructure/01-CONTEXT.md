# Phase 1: Core ASR Infrastructure - Context

**Gathered:** 2026-03-08
**Status:** Ready for planning

<domain>
## Phase Boundary

Users can trigger voice recording via CapsLock, see real-time recognition results, with proper punctuation. This includes WebSocket server, streaming ASR, and configuration.

</domain>

<decisions>
## Implementation Decisions

### ASR Integration
- Use Sherpa-ONNX for offline ASR (already integrated in existing code)
- Support multiple models: FunASR-Nano, Paraformer, SenseVoice
- Streaming mode with 25s segments and 2s overlap (from CLAUDE.md)
- Dual result merging: text (simple) and text_accu (token timestamp deduplication)

### Server Architecture
- WebSocket server in `util/server/` already exists
- ASR runs in separate subprocess (non-blocking)
- Client/Server (C/S) architecture as established in CLAUDE.md

### Punctuation
- Punct-CT-Transformer model for offline punctuation
- Support Chinese and English punctuation

### Configuration
- config.py in root directory (already exists)
- Hotkey threshold: 0.3s default
- Audio segment duration: 25s default
- Hot reload support for config files

### Claude's Discretion
- Exact real-time display implementation (toast vs UI)
- Model selection defaults
- Error handling specifics

</decisions>

<specifics>
## Specific Ideas

From existing CLAUDE.md documentation:
- "Real-time streaming via WebSocket"
- "Client listens to CapsLock. Press to start recording, release to submit"
- "Server handles dual results: text (simple) and text_accu (precision)"
- "Hot reload: Client monitors hot*.txt and LLM/*.py changes"

</specifics>

<code_context>
## Existing Code Insights

### Reusable Assets
- `util/server/server_ws_recv.py` - WebSocket receive handling
- `util/server/server_ws_send.py` - WebSocket send handling
- `util/server/server_recognize.py` - ASR recognition
- `util/server/server_init_recognizer.py` - Model initialization
- `util/server/text_merge.py` - Text merging algorithms
- `util/client/global_hotkey/global_hotkey.py` - Hotkey handling
- `config.py` - Configuration (already exists)

### Established Patterns
- C/S architecture with separate subprocess for ASR
- WebSocket for real-time streaming
- Sherpa-ONNX integration for offline models

### Integration Points
- Client connects to server via WebSocket
- Audio chunks sent from client to server
- Recognition results sent back in real-time

</code_context>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope

</deferred>

---

*Phase: 01-core-asr-infrastructure*
*Context gathered: 2026-03-08*
