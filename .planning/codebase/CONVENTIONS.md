# Coding Conventions

**Analysis Date:** 2026-03-08

## Naming Patterns

**Files:**
- Python modules: `snake_case.py` (e.g., `recorder.py`, `rag_fast.py`, `llm_client_pool.py`)
- Test files: `test_*.py` in model directories (e.g., `models/.../test.py`)
- Entry points: `core_*.py` for source, `start_*.py` for packaged (e.g., `core_client.py`, `start_server.py`)

**Classes:**
- PascalCase: `AudioRecorder`, `FastRAG`, `PhonemeIndex`, `ClientConfig`
- Use descriptive nouns: `AudioFileManager`, `TextOutput`, `ShortcutManager`

**Functions:**
- snake_case: `record_and_send()`, `get_candidates()`, `compute_score()`
- Private methods: `_send_message()`, `_score_candidates()` (prefixed with `_`)
- Async functions: `async def record_and_send()`

**Variables:**
- snake_case: `task_id`, `input_phonemes`, `hotword_count`
- Private variables: `_file_manager`, `_start_time` (prefixed with `_`)
- Constants (class-level): `SAMPLE_RATE: int = 16000` (PascalCase class names, UPPER_SNAKE for true constants)

**Types:**
- Type hints used throughout with `typing` module: `Optional[str]`, `List[Tuple[str, float]]`, `Dict[str, int]`
- Use `TYPE_CHECKING` for imports to avoid circular dependencies

## Code Style

**Formatting:**
- Indentation: 4 spaces (no tabs)
- Line length: No hard limit, but generally under 120 characters
- No automatic formatter configured (not detected)

**Linting:**
- No formal linting configuration detected
- Uses standard Python conventions

**Encoding:**
- All files start with `# coding: utf-8` or `#!/usr/bin/env python3`

**Import Organization:**
1. Standard library imports (`asyncio`, `os`, `sys`)
2. Third-party imports (`numpy`, `websockets`, `pynput`)
3. Local application imports (`from config import ...`, `from util.client import ...`)

**Path Aliases:**
- Not detected - uses relative imports from project root

## Error Handling

**Patterns:**
- Try-except blocks with specific exception handling:
```python
try:
    await websocket.send(json.dumps(message))
except websockets.ConnectionClosedError:
    # Handle specific WebSocket errors
except Exception as e:
    logger.error(f"Error message: {e}", exc_info=True)
```

- Generic exception catching at top level with logging
- Uses `exc_info=True` for full stack traces in logs

**Custom Exceptions:**
- Custom wrapper exceptions in `util/llm/llm_exceptions.py`:
  - `OpenAIErrorWrapper`
  - `AuthenticationErrorWrapper`
  - `RateLimitErrorWrapper`
  - `TimeoutErrorWrapper`
  - `ConnectionErrorWrapper`
  - `APIResponseErrorWrapper`

**Error Recovery:**
- Fallback patterns: `should_fallback_to_original()` determines whether to degrade gracefully
- Toast notifications for user-facing errors via `show_error_notification()`

## Logging

**Framework:** Python standard `logging` module with custom `Logger` class in `util/logger.py`

**Patterns:**
- Class-based logger management with singleton pattern:
```python
from util import get_logger
logger = get_logger('client')
```
- Per-module loggers: `logger = get_logger(__name__)` pattern
- Log levels: DEBUG, INFO, WARNING, ERROR, CRITICAL
- Rotating file handlers with date-stamped filenames: `client_YYYYMMDD.log`
- Format: `%(asctime)s.%(msecs)03d - %(name)s - %(levelname)s - [%(filename)s:%(lineno)d] - %(message)s`

**When to Log:**
- DEBUG: Detailed diagnostic information
- INFO: Normal operation events
- WARNING: Unexpected but handled situations
- ERROR: Exceptions with full tracebacks

## Comments

**When to Comment:**
- All modules have module-level docstrings explaining purpose
- Classes have docstrings with description, attributes, and methods
- Complex algorithms have inline comments (e.g., DP matrix computation in `rag_fast.py`)

**Docstrings:**
- Chinese language comments (per CLAUDE.md preference)
- Google-style or simple description format:
```python
"""
模块功能描述

Provides X class for managing Y.
"""
```

**In-Code Documentation:**
- Use Chinese for all comments and documentation (as per CLAUDE.md)
- Complex logic blocks preceded by explanatory comments (e.g., `# 1. 编码输入并获取候选`)

## Function Design

**Size:**
- Functions typically under 100 lines
- Complex functions broken into private helpers: `_score_candidates()`, `_python_distance()`

**Parameters:**
- Type hints for all parameters
- Default values where appropriate: `def search(self, input_phonemes: List[Phoneme], top_k: int = 10)`

**Return Values:**
- Type hints on return types: `def search(...) -> List[Tuple[str, float]]:`
- Early returns for edge cases: `if not input_phonemes: return []`

**Async:**
- Use `asyncio` for I/O-bound operations (WebSocket, audio streaming)
- `async def` / `await` patterns throughout client code

## Module Design

**Exports:**
- Explicit `__all__` lists in `__init__.py` files
- Example from `util/client/__init__.py`:
```python
__all__ = [
    'ClientState',
    'WebSocketManager',
    'AudioRecorder',
    ...
]
```

**Barrel Files:**
- Each subdirectory has `__init__.py` that exports public API
- Organized by feature: `util/client/audio/`, `util/hotword/`, `util/llm/`

**Module Organization:**
- Client code: `util/client/`
- Server code: `util/server/`
- Shared utilities: `util/`
- Configuration: `config.py` (root), constants in `util/constants.py`

## Special Patterns

**Configuration:**
- Two-tier config: `ServerConfig` and `ClientConfig` classes in `config.py`
- User-configurable options in root `config.py`
- Internal constants in `util/constants.py`

**State Management:**
- Singleton state via `ClientState` class
- Shared through dependency injection: `def __init__(self, state: 'ClientState')`

**Hot Reloading:**
- File watchers for config files (`hot.txt`, `LLM/*.py`)
- Pattern: Modify -> Watchdog detects -> Reload without restart

---

*Convention analysis: 2026-03-08*
