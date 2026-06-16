# AI-Assisted Project Setup: A Senior Engineer's Workflow

The exact session, top to bottom, for spinning up a new project with Claude Code
using best practices — `/brainstorm`, plan mode, evals, `AGENTS.md`,
`copilot-instructions.md`, and skills.

The running example is a FastAPI service where operators send a natural-language
query and get an answer from an LLM on Bedrock, but the steps are a reusable
template.

The whole sequence follows one principle: **think before you write at every
layer.** Steps 1–5 produce zero application code, and that is exactly why the code
that follows is good.

---

## 0. Setup

In an empty repo directory, start Claude Code on a strong planning model and
install the brainstorming plugin (it is not built-in):

```bash
claude --model opus
/plugin            # add Superpowers, which provides /brainstorming
```

---

## 1. Brainstorm — diverge before you converge

Don't ask for a plan yet; ask to explore.

> `/brainstorming` "I want a FastAPI service where plant-floor operators send a
> natural-language query and get an answer from an LLM on Bedrock. Help me surface
> constraints, hidden requirements, and 2–3 architectural options before we commit.
> Ask me questions."

**Output:** a signed-off design doc. This is the cheapest place to change your mind.

---

## 2. Plan mode — turn the design into an implementation plan (read-only)

Cycle into plan mode (`Shift+Tab` to `plan`, or `/plan`):

> "Based on the design doc, produce an implementation plan: endpoint contract,
> file layout, the Bedrock client boundary, error handling, and where evals will
> live. Don't write code."

Review and approve the plan before anything is allowed to write.

---

## 3. Cross-review the plan

Open a **second** Claude Code session and paste the plan:

> "Review this plan as a skeptical staff engineer. What assumptions are unstated?
> What breaks under load or bad input?"

Fold the catches back into the plan in session one.

---

## 4. Write `AGENTS.md` / `CLAUDE.md` — house rules before code

> "From the approved plan, write an `AGENTS.md`: stack, directory layout,
> conventions (all LLM calls go through `llm/client.py`; every endpoint gets a
> Pydantic request and response model), and how to run and test."

Then **you** hand-edit it. This is the one artifact you personally own.

---

## 5. Mirror to `copilot-instructions.md` — one source of truth

> "Generate `.github/copilot-instructions.md` from `AGENTS.md`, condensed to the
> conventions Copilot needs. Keep `AGENTS.md` canonical."

---

## 6. Scaffold a minimal runnable skeleton

Exit plan mode.

> "Create the project structure and a stub endpoint that returns a fixed response,
> following `AGENTS.md`. I want to `curl` it before any LLM logic."

---

## 7. Write the eval cases — before the prompt, not after

> "Before we write the system prompt, draft 6–8 eval cases: realistic operator
> queries plus, for each, what a good answer must satisfy (format, refuses
> out-of-scope, stays on topic). Save them as an eval set for me to sign off."

Writing these first means you build the prompt *toward a target*.

---

## 8. Build the LLM layer and run it against the evals

> "Implement the Bedrock client and system prompt, run them against the eval set,
> and show me which cases fail and why."

Iterate on `prompts.py` using the failures as your signal — this is the loop where
the prompt gets good.

---

## 9. `pytest` for the deterministic plumbing — kept separate

> "Add pytest: 200 on valid input, 422 on malformed, auth rejection, Bedrock client
> called with the right args (mocked)."

---

## 10. Wire CI to run both

> "Add a CI workflow that runs pytest and the eval set on PRs, failing the build if
> either regresses."

---

## 11. (Later, only when a pattern repeats) Capture it as a skill

After you've added the third query type the same way:

> "Use the skill-creator. Turn 'add a new query type — Pydantic model + prompt
> template + matching eval' into a skill. Run the eval loop (with-skill vs
> baseline), show me the review viewer, optimize the trigger description, and
> package the `.skill`."

This is the side-loop: the skill's **own** evals, not the project's. The finished
skill then feeds every future build.

---

## The two eval loops, kept straight

| Loop | What it measures | When it runs |
|------|------------------|--------------|
| **Project LLM evals** | Your endpoint's prompt output | Inside every project (step 7–8) |
| **Skill-creator evals** | Whether the skill itself works and triggers | Only when authoring a skill (step 11) |

Same word, two layers — and they nest: a skill that *generates* endpoints is
checked by skill-creator evals, and each endpoint it generates then gets its own
project LLM evals.