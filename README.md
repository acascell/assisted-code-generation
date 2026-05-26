# Chatbot

Conversational chatbot with a Streamlit UI backed by a FastAPI + LangChain agent.

```
frontend/   Streamlit UI  →  http://localhost:8501
backend/    FastAPI + LangChain  →  http://localhost:8000
```

---

## Quick Start (Docker Compose)

```bash
# 1. Copy and fill in env files
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env
# Edit backend/.env and add your OPENAI_API_KEY

# 2. Build and run
docker compose up --build

# App: http://localhost:8501
# API: http://localhost:8000/docs
```

---

## Local Development

### Backend

```bash
cd backend
pip install -r requirements.txt
cp .env.example .env   # then add your OPENAI_API_KEY
python main.py
# API running at http://localhost:8000
# Interactive docs at http://localhost:8000/docs
```

### Frontend

```bash
cd frontend
pip install -r requirements.txt
cp .env.example .env   # BACKEND_URL=http://localhost:8000
streamlit run app.py
# UI at http://localhost:8501
```

---

## Testing

```bash
# Backend unit tests (no API calls)
cd backend && pytest tests/unit/ -v

# Backend integration tests (requires .env with valid key)
cd backend && pytest tests/integration/ -v

# With coverage
cd backend && pytest --cov=. --cov-report=term-missing
```

---

## Project Layout

```
├── backend/
│   ├── agents/             LangChain AgentExecutor
│   ├── chains/             LCEL chains
│   ├── tools/              Custom LangChain tools (one file per tool)
│   ├── memory/             Per-session ConversationBufferMemory store
│   ├── api/                FastAPI routes and Pydantic schemas
│   ├── config/             Pydantic BaseSettings (single source of truth)
│   ├── tests/
│   │   ├── unit/           Mocked — no LLM calls
│   │   └── integration/    Real API calls, requires .env
│   └── main.py             FastAPI entry point
│
├── frontend/
│   ├── app.py              Streamlit entry point (UI only)
│   ├── components/         Reusable Streamlit components
│   ├── utils/api_client.py HTTP client to backend
│   └── config/             Frontend settings
│
├── docker-compose.yml
├── AGENTS.md               AI assistant context (read this first)
└── CLAUDE.md               Claude Code config
```

---

## Key Constraints

- `app.py` contains only rendering logic — no business logic
- All LLM/chain objects use `@lru_cache` or `@st.cache_resource` — never module-level
- Config via `config/settings.py` only — no `os.environ` calls in app code
- LCEL chains only — no legacy `LLMChain` or `ConversationalRetrievalChain`
- Unit tests never make real API calls
