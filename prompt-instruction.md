# prompt 1
I'm building a [description of your project].

Tech stack:
- [language + version]
- [frameworks]
- [infra: Lambda, ECS, etc.]
- [key constraints: timeouts, auth, etc.]

Generate for me:
1. AGENTS.md — cross-tool project context
2. .github/copilot-instructions.md — Copilot-specific rules
3. CLAUDE.md — Claude Code config

Include: architecture overview, naming conventions,
anti-patterns to avoid, how to run/test/build,
and any non-obvious constraints.

# prompt 2

Based on AGENTS.md, scaffold the initial directory
structure and placeholder files. Include:
- src/ layout
- tests/ mirroring src/
- CI config (GitHub Actions)
- pyproject.toml / package.json with agreed deps
- .github/instructions/ stubs for each layer


# prompt 3
Based on AGENTS.md, generate:
1. Pydantic v2 models for all core domain entities
2. API contract (OpenAPI 3.0 or typed interfaces)
3. Key abstractions / protocols (e.g. repository interfaces)

No implementation. Types and shapes only.

# prompt 4
Generate pytest test stubs for each model and interface.
Tests should verify: validation rules, edge cases,
and the contract boundaries. Use our test conventions
from AGENTS.md.

# prompt 5
Implement [specific module] to pass the tests in
tests/[path]. Follow the patterns in AGENTS.md.
Do not modify the test files.

Instructions → Skeleton → Contracts → Tests → Implementation


# prompt 1.1
I'm building a manufacturing analytics REST API.

Stack:
- Python 3.12, FastAPI BFF, Lambda handlers
- DynamoDB, AWS Bedrock, LangGraph 0.2.x
- Cognito JWT auth, API Gateway (29s timeout)
- Poetry, pytest + moto, CDK for infra

Constraints:
- No global state in Lambda
- All agent calls must stream (timeout workaround)
- Pydantic v2 for all I/O boundaries
- boto3 clients instantiated per-handler

Generate: AGENTS.md, CLAUDE.md,
.github/copilot-instructions.md,
and path-specific stubs for:
- src/handlers/
- src/agents/
- tests/
- cdk/

# prompt 1.2
Based on AGENTS.md, scaffold the full monorepo
directory structure with empty placeholder files.

Include:
- frontend/ (React app, Vite)
- backend/ (FastAPI, Poetry)
- infra/ (CDK stack)
- .github/workflows/ (CI for both frontend and backend)
- docker-compose.yml for local dev
- README.md

# prompt 1.3
Enhance the current structure defining skills
