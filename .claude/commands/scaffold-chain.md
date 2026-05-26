# scaffold-chain

Create a new LCEL chain named **$ARGUMENTS** following this project's conventions.

## Steps

1. **Create `backend/chains/$ARGUMENTS_chain.py`**
   - Define a factory function `build_$ARGUMENTS_chain() -> Runnable`
   - Use pipe syntax: `prompt | llm | parser`
   - Get the LLM from `config/settings.py` — never hardcode model names
   - Use `ChatPromptTemplate.from_messages([...])` with a `MessagesPlaceholder` for `chat_history` if the chain is conversational
   - Parser: `StrOutputParser` for plain text, or a Pydantic output parser for structured output

2. **Export from `backend/chains/__init__.py`** if the module needs to be importable by name

3. **Create `backend/tests/unit/test_$ARGUMENTS_chain.py`**
   - Mock the LLM with `RunnableLambda(lambda _: AIMessage(content="ok"))` 
   - Verify the chain invokes and returns the right shape
   - No real API calls

## Constraints from AGENTS.md
- LCEL only (`|` pipe) — never `LLMChain` or legacy chain classes
- Chain file must be named `$ARGUMENTS_chain.py`
- Factory function, not module-level chain variable
- Always set `timeout` on LLM constructors
