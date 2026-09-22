# Constitution for Saylani Student Ops Desk

## Model Configuration
- The Desk uses **gemini-2.5-flash** through an OpenAI-compatible endpoint.
- Model is configured **per-agent** at `Agent(model=...)`. No global or per-run defaults.
- `set_default_openai_client` is **forbidden**.

## Secrets
- Secrets live **only in `.env`**, which is **gitignored**.
- A **missing key produces a clear startup error**, not a stack trace three layers deep.

## Tools
- Tools **never raise** to the caller; they return **actionable sentences** the model can act on.
- A tool that hits bad data returns a sentence the model can act on (NFR-4).
- A tool that raises into the runner is a **defect**, not a feature (NFR-4).

## Student Data & Privacy
- **No student data reaches the model** except via deliberately written instructions/tools.
- `StudentProfile` travels via **run context**, **never** in prompt text.
- The prompt text never contains the student's name, roll number, or tier.

## Agent Model Declarations
- **Every agent declares its own model settings**.
- No agent inherits model configuration implicitly; each explicitly states `Agent(model=...)`.

## Entry Point
- The entry point is **async** via **`asyncio.run`**.

## Cut Order (When Behind Schedule)
If behind schedule, cut **only** in this order:
1. FR-11
2. FR-6
3. **AgentHooks half of** FR-10

**FR-7 and FR-8 are never cut** — the guardrail and the ticket are what the viva is built on.

## Four Absolute Rules (from Phase 0)
1. Model provider and configuration as specified above.
2. Secrets live only in `.env`.
3. Tools never raise to the caller.
4. No student data reaches the model except through deliberately written instructions/tools.