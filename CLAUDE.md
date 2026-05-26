# CLAUDE.md

Configuration and context for Claude Code working in this repository.

---

## Project

Python 3.11+ chatbot — Streamlit UI + LangChain (LCEL) agents. See `AGENTS.md` for the full architecture reference.

---

## Commands

```bash
# Run the app
streamlit run app.py

# Run all tests
pytest

# Unit tests only (no API calls)
pytest tests/unit/

# Lint
ruff check .

# Type check
mypy . --ignore-missing-imports

# Install deps
pip install -r requirements.txt
```

---

## Code Conventions

- Type hints on all function signatures, including return types
- `snake_case`: files, functions, variables, session state keys
- `PascalCase`: classes
- `UPPER_SNAKE_CASE`: module-level constants
- Tool files: `<verb>_<noun>_tool.py`; Chain files: `<name>_chain.py`; Agent files: `<name>_agent.py`
- No comments that restate what the code does — only comments explaining non-obvious constraints or workarounds

---

## Architecture Rules (enforce these when editing)

1. `app.py` is UI-only. If you find business logic there, move it to `agents/` or `chains/`.
2. Every LLM, chain, or agent must be constructed inside a `@st.cache_resource`-decorated factory function.
3. All conversation history and memory objects live in `st.session_state` — never in module-level variables.
4. Config is accessed through `config/settings.py` only. If you need a new env var, add it to `Settings` and `.env.example`.
5. Use LCEL (`prompt | llm | parser`) everywhere. Do not introduce legacy `LLMChain` or `ConversationalRetrievalChain`.

---

## Non-Obvious Constraints

**Streamlit reruns the entire script on every interaction.** Any object created outside `@st.cache_resource` or `st.session_state` will be recreated on every user message. This is the most common source of bugs in this stack — always check where state is stored.

**LangChain memory cannot be cached with `@st.cache_resource`.** Cache the chain; store the memory in `st.session_state`. Reason: `st.cache_resource` shares the object across sessions in multi-user deployments; memory must be per-session.

**No async at the Streamlit top level.** Streamlit's script runner is synchronous. Use sync LangChain methods (`invoke`, not `ainvoke`) in `app.py`. If async is needed in a tool, isolate it behind a sync wrapper using `asyncio.run()`.

**Tool errors bubble to the agent by default.** LangChain agents catch `ToolException` and include the error in the agent scratchpad. Raise `ToolException` (not bare `Exception`) in tool `_run` methods so the agent can reason about failures.

---

## Testing Guidelines

- Mock LLMs in unit tests using `unittest.mock.patch` or `RunnableLambda(lambda x: ...)` — no real API calls
- Integration tests (under `tests/integration/`) may call real APIs and require a populated `.env`
- Run `pytest tests/unit/` in CI; gate `tests/integration/` behind an explicit flag or environment check
- Streamlit components are tested by exercising the underlying chain/agent logic directly, not by driving the UI

---

## Anti-Patterns

| Pattern | Problem |
|---|---|
| Logic in `app.py` | Untestable, couples UI to LLM |
| LLM at module level | Recreated every rerun, leaks connections |
| `os.environ` outside settings | No validation, breaks in tests |
| Legacy chain classes | Deprecated, inconsistent interface |
| Memory inside `@st.cache_resource` | Shared across users, wrong per-session state |
| Bare `except Exception` in tools | Swallows LangChain internal errors |
| Hardcoded model name strings | Hard to swap or test with cheaper models |

---

## Files to Never Edit Directly

- `.env` — managed by the user, not committed
- `requirements.txt` — update via `pip freeze` after installing packages

---

## Security

- API keys in `.env` only, loaded by `config/settings.py`
- `.env` is in `.gitignore` — never commit it
- User input is passed to the LangChain agent as a string — never interpolated into raw SQL, shell commands, or prompts that skip the agent's input sanitisation layer
- Do not log full message content at INFO level — it may contain PII
