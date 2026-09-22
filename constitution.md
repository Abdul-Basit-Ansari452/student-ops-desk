# Constitution for Saylani Student Ops Desk

## Model Configuration

- The Desk runs on **gemini-2.5-flash** through an OpenAI-compatible client.
- The model is set **per-agent** at `Agent(model=...)` — never globally, never per-run via `set_default_openai_client`.
- `set_default_openai_client` is **forbidden** in any agent definition.
- Every agent **declares its own model settings** explicitly; no global default overrides them.

## Secrets and Configuration

- All secrets (API keys, endpoints) live **only in `.env`**, which is **gitignored**.
- A **missing key produces a clear startup error** at program initiation, not a stack trace three layers deep.
- If a required key is absent, the program exits immediately with a clear message.

## Student Data and Privacy

- **No student data reaches the model** except via deliberately written instructions and tools.
- `StudentProfile` travels via **run context**, never as prompt text. The prompt never contains the student's name, roll number, or tier.
- A `StudentProfile` dataclass contains: `name`, `roll_no`, `course_id`, `tier` (default "regular"), `open_tickets` (default 0).
- The system prompt is built at request time from the profile: greets by name, names the course, becomes terser when `open_tickets >= 3`.

## Tools and Execution

- **Tools never raise to the caller**. They return **actionable sentences** or structured data the runner can act on.
- A tool that hits bad data returns a sentence the model can act on; raising into the runner is a defect.
- Course facts live in `courses.json`; the agent can only reach them by calling tools.
- Tools are **offered only to students whose tier is scholarship**; absent (not refused) for everyone else.

## conversation and Run Lifecycle

- The entry point is an **asynchronous function driven by `asyncio.run`**.
- A **custom runner** stamps a request ID and elapsed time around every run, registered once at startup.
- No agent definition changes to accommodate the runner.
- Run-level hooks record an **ordered timeline** covering every agent in a conversation, including handoffs.
- Agent-level hooks are attached to exactly one specialist and go quiet at the moment the handoff happens.

## Structural Constraints (Cut Order)

If the project falls behind, cut features in this order (first to last):

1. **FR-11** (Custom runner)
2. **FR-6** (Summariser as tool)
3. **AgentHooks half of FR-10** (agent-level hooks for the specialist only)

**Never cut** FR-7 (structured ticket) or FR-8 (guardrail refusing non-course questions). These are the guardrail and the ticket — the viva is built on them.

## Observability

- Every conversation is **traceable**; the audit timeline from FR-10 is written somewhere durable, not only printed.
- Tracing is on, exported under your own key, and a single student conversation appears as one trace rather than several.
- One span per agent per model call; the run timeline names every agent in order.

## What the Build Will Not Do

1. Use `set_default_openai_client` anywhere.
2. Send student data to the model outside of deliberate instructions/tools.
3. Reach course data without tool calls.
4. Generate without a model ceiling (every agent declares its own settings).