# scaffold-tool

Create a new LangChain tool named **$ARGUMENTS** following this project's conventions.

## Steps

1. **Create `backend/tools/$ARGUMENTS_tool.py`**
   - Subclass `BaseTool` from `langchain_core.tools`
   - Class name: PascalCase of `$ARGUMENTS` + `Tool` (e.g. `SearchTool`)
   - `name` field: `$ARGUMENTS` in snake_case
   - `description` field: one clear sentence — the agent uses this to decide when to call the tool
   - Implement `_run(self, query: str) -> str` (sync)
   - Raise `ToolException` (never bare `Exception`) on failure
   - Type-hint every method signature

2. **Register the tool** in `backend/agents/chat_agent.py` inside `_get_tools()` — append an instance of the new tool class to the returned list

3. **Create `backend/tests/unit/test_$ARGUMENTS_tool.py`**
   - One happy-path test verifying `_run` returns the expected string
   - One error-path test verifying `ToolException` is raised on failure
   - Mock all external I/O — no real API calls

## Constraints from AGENTS.md
- Never catch bare `Exception` — use `ToolException`
- Tool file must be named `$ARGUMENTS_tool.py` (snake_case)
- No business logic in `app.py`
- All config via `config/settings.py`
