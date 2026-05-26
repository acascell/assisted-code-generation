# AGENTS.md — Project Context for AI Coding Assistants

Cross-tool reference for Claude, Copilot, Cursor, and any other AI assistant working in this repo.

---

## Project Purpose

A conversational chatbot application with an interactive Streamlit UI backed by LangChain agents.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Python 3.11+ |
| UI | Streamlit |
| AI Orchestration | LangChain (LCEL-first) |
| LLM | Configurable via `config/settings.py` (default: OpenAI) |
| Memory | LangChain `ConversationBufferMemory` stored in `st.session_state` |
| Config | `python-dotenv` + `.env` |
| Testing | `pytest` + `pytest-mock` |

---

## Repository Layout

```
├── app.py                        # Streamlit entry point — only UI logic here
├── agents/
│   ├── __init__.py
│   └── chat_agent.py             # LangChain agent definition (AgentExecutor)
├── chains/
│   ├── __init__.py
│   └── conversation_chain.py     # LCEL chains (RunnableSequence)
├── tools/
│   ├── __init__.py
│   └── <tool_name>_tool.py       # One file per custom LangChain tool
├── memory/
│   ├── __init__.py
│   └── session_memory.py         # Helpers to bind memory to st.session_state
├── config/
│   └── settings.py               # Pydantic BaseSettings — single source of truth
├── utils/
│   └── callbacks.py              # Streamlit-compatible streaming callbacks
├── tests/
│   ├── unit/
│   └── integration/
├── .env.example                  # Committed template, never commit .env
├── requirements.txt
└── pyproject.toml
```

---

## Architecture Overview

```
User input (Streamlit UI)
    │
    ▼
st.session_state.messages  ←─────────────────────────────┐
    │                                                      │
    ▼                                                      │
AgentExecutor (agents/chat_agent.py)                      │
    │                                                      │
    ├── LLM (via config/settings.py)                       │
    ├── Tools (tools/*.py)                                 │
    └── Memory (ConversationBufferMemory) ─────────────────┘
            stored in st.session_state
```

Key flow: `app.py` owns rendering and session state. It calls into `agents/` or `chains/` but **never** contains business logic. Agents/chains are stateless Python objects; all conversation state lives in `st.session_state`.

---

## Naming Conventions

### Files & Modules
- `snake_case` for all Python files and directories
- Suffix tool files with `_tool.py` (e.g., `search_tool.py`, `calculator_tool.py`)
- Suffix chain files with `_chain.py`
- Suffix agent files with `_agent.py`

### Python
- `snake_case` for functions, variables, module names
- `PascalCase` for classes
- `UPPER_SNAKE_CASE` for constants and environment variable names
- Prefix private helpers with a single underscore (`_build_prompt`)
- LangChain `Runnable` variables: descriptive noun phrases (`conversation_chain`, `retrieval_chain`)

### Streamlit
- Session state keys: `snake_case` string literals (e.g., `st.session_state["chat_history"]`)
- Cached resources: decorate factory functions with `@st.cache_resource`, not the class directly

---

## Non-Obvious Constraints

### Streamlit reruns on every interaction
Streamlit re-executes `app.py` top-to-bottom on **every user action**. This means:
- Never instantiate LLMs, chains, or agents at module level — use `@st.cache_resource`.
- Never hold conversation history in a plain Python variable — use `st.session_state`.
- `@st.cache_data` is for serialisable data (strings, dicts). `@st.cache_resource` is for objects with connections or state (LLMs, chains, vector stores).

### LangChain memory must live in session_state
`ConversationBufferMemory` (and all LangChain memory classes) must be stored in `st.session_state`, not created inside a cached resource. Cache the chain; store the memory separately.

```python
# CORRECT
@st.cache_resource
def get_llm():
    return ChatOpenAI(model=settings.model_name)

if "memory" not in st.session_state:
    st.session_state["memory"] = ConversationBufferMemory(return_messages=True)
```

### Prefer LCEL over legacy chains
Use `RunnableSequence` / pipe syntax (`|`) instead of `LLMChain`, `ConversationalRetrievalChain`, or other legacy classes. Legacy chain classes are deprecated as of LangChain 0.2.

### Async is not natively supported in Streamlit
Streamlit runs synchronously. Do not use `async def` handlers or `await` at the top level of `app.py`. If an async LangChain method must be used, wrap it with `asyncio.run()` inside a sync function — never mix awaitable and non-awaitable paths.

### Environment variables
- All secrets and model names go in `.env` (never committed)
- `.env.example` documents every required key with a placeholder value
- Access config exclusively through `config/settings.py` (Pydantic `BaseSettings`) — never call `os.environ` directly outside that module

---

## Anti-Patterns to Avoid

| Anti-pattern | Why | Do instead |
|---|---|---|
| Business logic in `app.py` | Couples UI to logic, kills testability | Put logic in `agents/` or `chains/` |
| LLM instantiated at module level | Reinitialised on every rerun, leaks connections | Wrap in `@st.cache_resource` |
| `os.environ["KEY"]` scattered in code | No validation, silent failures | Use `config/settings.py` |
| Plain `dict` for chat history in a variable | Lost on rerun | `st.session_state["messages"]` |
| Legacy `LLMChain` / `ConversationalRetrievalChain` | Deprecated, inconsistent interface | LCEL `RunnableSequence` |
| Hardcoded model name strings | Hard to swap models | `settings.model_name` from config |
| Catching bare `Exception` in tool code | Hides LangChain internal errors | Catch specific exceptions; let others propagate |
| Synchronous blocking calls inside `st.spinner` without error handling | Silent hang if LLM times out | Always set `request_timeout` and handle `TimeoutError` |
| Storing full message objects in memory AND session_state | Double source of truth | Memory is for the chain; session_state is for rendering |

---

## How to Run

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Copy and fill in secrets
cp .env.example .env

# 3. Start the app
streamlit run app.py

# Optional: custom port
streamlit run app.py --server.port 8502
```

---

## How to Test

```bash
# All tests
pytest

# Unit tests only (no LLM calls)
pytest tests/unit/

# Integration tests (requires .env with valid API keys)
pytest tests/integration/

# With coverage
pytest --cov=. --cov-report=term-missing

# Single file
pytest tests/unit/test_chat_agent.py -v
```

Mock LLM calls in unit tests using `langchain_core.runnables.base.RunnableLambda` or `unittest.mock.patch` on the LLM class. Never make real API calls in unit tests.

---

## How to Build / Package

```bash
# Freeze dependencies
pip freeze > requirements.txt

# Lint
ruff check .

# Type check
mypy . --ignore-missing-imports
```

There is no compile step. For containerised deployment:
```bash
docker build -t chatbot .
docker run -p 8501:8501 --env-file .env chatbot
```
