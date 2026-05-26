# add-route

Add a new FastAPI route for **$ARGUMENTS** following this project's conventions.

## Steps

1. **Add request/response schemas to `backend/api/schemas.py`**
   - `<PascalCase>Request(BaseModel)` with typed fields and `Field(...)` annotations
   - `<PascalCase>Response(BaseModel)` — never return raw dicts from endpoints

2. **Add the route to `backend/api/routes.py`** (or a new router file if the domain is distinct)
   - Use the appropriate HTTP verb
   - Annotate the return type with the response model: `response_model=<Name>Response`
   - Catch `TimeoutError` → 504, unexpected errors → 500 with `HTTPException`
   - Delegate all logic to `agents/` or `chains/` — no business logic in the route handler

3. **Register a new router** in `backend/main.py` if you created a separate router file

4. **Create `backend/tests/unit/test_<route_name>_route.py`**
   - Use FastAPI `TestClient`
   - Mock the agent/chain call — no real LLM calls
   - Test happy path and at least one error path (e.g., downstream timeout)

## Constraints from AGENTS.md
- Route handlers contain no business logic
- Always use typed Pydantic schemas for I/O
- All config from `config/settings.py`
