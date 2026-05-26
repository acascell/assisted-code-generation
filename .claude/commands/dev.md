# dev

Start the full development environment locally.

## Pre-flight checks

1. Verify `backend/.env` exists — if not, run: `cp backend/.env.example backend/.env` and fill in `OPENAI_API_KEY`
2. Verify `frontend/.env` exists — if not, run: `cp frontend/.env.example frontend/.env`
3. Verify dependencies are installed in both services

## Start services

Run these two commands in separate terminals (or background processes):

```bash
# Terminal 1 — Backend (FastAPI)
cd backend && python main.py
# Listening at http://localhost:8000
# Docs at http://localhost:8000/docs

# Terminal 2 — Frontend (Streamlit)
cd frontend && streamlit run app.py
# UI at http://localhost:8501
```

## Alternatively — Docker Compose

```bash
docker compose up --build
```

## Health check

After both services start, verify the stack is wired correctly:
```bash
curl http://localhost:8000/health
# Expected: {"status": "ok"}
```

Then open http://localhost:8501 and send a test message.

## Common issues

| Symptom | Cause | Fix |
|---|---|---|
| Frontend shows "Error: Connection refused" | Backend not running | Start backend first |
| Backend 500 on `/chat` | Missing or invalid `OPENAI_API_KEY` | Check `backend/.env` |
| Streamlit reloads on every keystroke | Normal behavior — state lives in `st.session_state` | Not a bug |
| Backend timeout | `REQUEST_TIMEOUT` too low for the model | Increase in `backend/.env` |
