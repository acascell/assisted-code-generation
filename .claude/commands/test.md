# test

Run the test suite for this project. Optionally pass a scope as **$ARGUMENTS**.

## Scopes

| Argument | Command | Notes |
|---|---|---|
| (none) | `pytest backend/tests/unit/` | Default: unit tests, no API calls |
| `unit` | `pytest backend/tests/unit/ -v` | Verbose unit tests |
| `integration` | `pytest backend/tests/integration/ -v` | Requires valid `backend/.env` |
| `coverage` | `pytest backend/tests/unit/ --cov=backend --cov-report=term-missing` | Coverage report |
| `<filename>` | `pytest backend/tests/unit/test_$ARGUMENTS.py -v` | Single test file |

## Rules enforced in this project (from AGENTS.md)

- Unit tests must never make real API calls — mock LLMs with `RunnableLambda` or `unittest.mock.patch`
- Test files mirror source: `backend/agents/chat_agent.py` → `backend/tests/unit/test_chat_agent.py`
- Integration tests are gated from CI unless explicitly triggered
- Use `pytest` fixtures for shared setup — no copy-paste setup blocks

## Run now

```bash
cd backend && pytest tests/unit/ -v
```

If any test imports fail, ensure dependencies are installed:
```bash
pip install -r backend/requirements.txt
```
