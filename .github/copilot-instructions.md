# GitHub Copilot Instructions

Context for Copilot suggestions in this repo. Read alongside `AGENTS.md` for the full picture.

---

## Project

Python + Streamlit chatbot application using LangChain (LCEL). Python 3.11+.

---

## Code Style

- PEP 8, enforced by `ruff`
- Type hints on all function signatures
- No bare `except:` — always name the exception
- No mutable default arguments
- Imports: stdlib → third-party → local, separated by blank lines

---

## Streamlit Rules

- All LLM/chain/agent instantiation must be inside a function decorated with `@st.cache_resource`
- Conversation state goes in `st.session_state`, keyed with `snake_case` strings
- `app.py` contains only rendering logic and `st.session_state` management — no business logic
- Do not use `async def` at the top level of `app.py`

**Example: correct Streamlit + LangChain wiring**
```python
@st.cache_resource
def get_chain() -> Runnable:
    llm = ChatOpenAI(model=settings.model_name, temperature=0)
    prompt = ChatPromptTemplate.from_messages([...])
    return prompt | llm | StrOutputParser()

if "messages" not in st.session_state:
    st.session_state["messages"] = []

if "memory" not in st.session_state:
    st.session_state["memory"] = ConversationBufferMemory(return_messages=True)
```

---

## LangChain Rules

- Use LCEL (`|` pipe syntax, `RunnableSequence`) — not legacy `LLMChain` or `ConversationalRetrievalChain`
- Custom tools must subclass `BaseTool` and implement `_run` (sync) and optionally `_arun` (async)
- Tool names: `snake_case`, descriptive verb-noun phrases (e.g., `search_web`, `get_weather`)
- Always set `request_timeout` on LLM constructors
- Bind memory to the chain using `RunnableWithMessageHistory`, not by mutating memory objects directly

---

## Configuration

- All settings come from `config/settings.py` (Pydantic `BaseSettings`)
- Never call `os.environ` outside `config/settings.py`
- Never hardcode model names, API keys, or URLs

---

## Testing

- Unit tests mock the LLM with `RunnableLambda` or `unittest.mock.patch` — never call real APIs
- Test file names mirror source: `tools/search_tool.py` → `tests/unit/test_search_tool.py`
- Use `pytest` fixtures for shared setup (LLMs, chains, session state stubs)

---

## What NOT to suggest

- `LLMChain`, `ConversationalRetrievalChain`, or any `from langchain.chains` import — these are deprecated
- `st.experimental_*` APIs — use the stable equivalents
- `os.getenv` / `os.environ` calls outside `config/settings.py`
- Global mutable state outside `st.session_state`
- `print()` for debugging — use `logging`
