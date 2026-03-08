# Testing Patterns

**Analysis Date:** 2026-03-08

## Test Framework

**Framework:** Not formally configured

This project does not use a formal testing framework (pytest, unittest). Testing appears to be ad-hoc:

- No `pytest.ini`, `setup.cfg`, or `pyproject.toml` with test configuration
- No test discovery pattern in place

**Test File Location:**
- Only one test file exists: `models/Punct-CT-Transformer/.../test.py`
- This is a model integration test, not a unit test

**Run Commands:**
```bash
# No formal test runner configured
python models/Punct-CT-Transformer/.../test.py    # Manual model test
python util/hotword/rag_fast.py                 # Standalone module test (via __main__)
```

## Test File Organization

**Location:**
- No standardized test directory
- Tests embedded in modules via `if __name__ == "__main__":` blocks

**Naming:**
- Test modules: `test.py` in model directories
- Inline tests: `if __name__ == "__main__":` blocks in modules

**Structure Example:**
```python
# In util/hotword/rag_fast.py
if __name__ == "__main__":
    # Test code here
    import random
    from util.hotword.algo_phoneme import get_phoneme_seq

    print(f"\n=== 高性能 RAG 测试 ===")
    # Test execution code
```

## Test Structure

**Pattern 1: Inline Module Tests**
Modules include self-test code in `if __name__ == "__main__":` blocks:

```python
# From util/hotword/rag_fast.py
if __name__ == "__main__":
    import random
    from util.hotword.algo_phoneme import get_phoneme_seq

    print(f"\n=== 高性能 RAG 测试 ===")
    print(f"Numba 可用: {HAS_NUMBA}")

    # Generate test data
    chinese_chars = '...'
    hotwords = {}
    for i in range(10000):
        word = ''.join(random.choice(chinese_chars) for _ in range(length))
        phonemes = get_phoneme_seq(word)
        hotwords[word] = phonemes

    # Run tests
    rag = FastRAG(threshold=0.6)
    rag.add_hotwords(hotwords)
    # Verify results...
```

**Pattern 2: Model Integration Tests**
Model directories contain test scripts:

```python
# From models/Punct-CT-Transformer/.../test.py
def main():
    text = "你好吗how are you我很好谢谢"
    model = OnnxModel()
    result = model(text)
    print(result)

if __name__ == "__main__":
    main()
```

## Mocking

**Mocking Framework:** Not used

No mocking framework (unittest.mock, pytest-mock) is used in this codebase because:
- No formal unit tests exist
- Integration tests run with real components

**What to Mock (Recommendations for Future):**
- WebSocket connections (in `util/client/websocket_manager.py`)
- Audio input streams (in `util/client/audio/recorder.py`)
- LLM API responses (in `util/llm/`)
- File system operations

**What NOT to Mock:**
- Hotword RAG algorithms (test with real data)
- Model inference (integration test with actual ONNX models)
- Numba-accelerated functions (test performance separately)

## Fixtures and Factories

**Test Data:**
- Generated inline in test blocks using random data:
```python
chinese_chars = '的一是不了在人有我他这个们中来上大为和国地到以说时要就出会可也你对生能而子那得于着下自之年过发后作里如等'

for i in range(10000):
    length = random.randint(2, 4)
    word = ''.join(random.choice(chinese_chars) for _ in range(length))
```

- Model test uses hardcoded samples:
```python
text = "你好吗how are you我很好谢谢"
```

**Location:**
- No dedicated fixtures directory
- Test data generated inline or uses existing config files (`hot.txt`, `hot-rule.txt`)

## Coverage

**Requirements:** None enforced

No code coverage tool configured or used.

**View Coverage:**
```bash
# No coverage command available
```

## Test Types

**Unit Tests:**
- Not currently implemented
- Inline tests in modules verify basic functionality
- Example: `rag_fast.py` tests the `FastRAG` class with generated data

**Integration Tests:**
- Model test (`models/.../test.py`) runs inference with ONNX model
- Manual testing via running `core_client.py` or `core_server.py`

**E2E Tests:**
- Not used
- Manual testing required for full workflow (CapsLock -> recording -> transcription -> output)

## Common Patterns

**Async Testing:**
- Not formally tested with pytest-asyncio
- Manual verification by running async functions

**Error Testing:**
- Custom exception wrappers in `util/llm/llm_exceptions.py` provide error categorization
- Error handling tested via manual triggers (disconnect network, invalid API keys)

## Recommendations

**Missing Testing Infrastructure:**
1. Add `pytest` as dev dependency
2. Create `tests/` directory structure:
   ```
   tests/
   ├── unit/
   │   ├── test_rag_fast.py
   │   ├── test_hotword.py
   │   └── test_llm_error_handler.py
   ├── integration/
   │   └── test_model_inference.py
   └── conftest.py
   ```

3. Add test fixtures for common data (hotwords, audio samples)

4. Configure CI/CD to run tests automatically

**Test Coverage Gaps:**
| Area | Status | Risk |
|------|--------|------|
| RAG algorithm | Inline only | Medium - performance regressions |
| Hotword matching | Manual | High - accuracy issues undetected |
| LLM integration | Manual | High - API errors not caught |
| WebSocket communication | Manual | High - connection issues |
| Audio recording | Manual | High - recording failures |
| Text processing | Manual | Medium - edge cases |
| Config loading | Not tested | Low - startup failures |

---

*Testing analysis: 2026-03-08*
