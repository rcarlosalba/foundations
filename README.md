# SDD Framework: How to Use

This framework transforms AI from a simple assistant into a structured engineering team. The Specification (Spec) is the only source of truth. Designed for Django, NestJS, and Next.js projects.

---

## Directory Structure

| Path | Purpose |
|------|---------|
| `CLAUDE.md` | Orchestrator — auto-loaded entry point for Claude Code |
| `0-prompts/` | Agent system prompts (Orchestrator, Coder, Tester, Validator) |
| `1-domain/` | Business concepts — technology-agnostic entity definitions |
| `2-rules/` | Tech stack standards (Django, NestJS, Next.js, Tailwind, Git) |
| `3-specs/` | Feature blueprints — what we're building right now |
| `4-context/` | Project memory (`project-map.md`, `decisions.log`, `archive/`) |
| `5-skills/` | Procedure runbooks for critical tasks (API integration, migrations, error handling) |

---

## Daily Workflow (The SDD Cycle)

### Step 0 — Define the Domain (when building new entities)

If the module introduces entities that don't exist yet, copy `1-domain/templates/entity-template.md` and define them first — in pure business terms, no technology. This feeds into the Spec.

### Step 1 — Write the Spec

Never request code without a Spec. Copy `3-specs/templates/spec-template.md` into your project and fill it in completely — including the **Out of Scope** section.

> **Golden Rule:** If it's not in the Spec, it doesn't exist. If the Spec changes, the code is regenerated.

### Step 2 — Initialize the Orchestrator

**Claude Code:** The Orchestrator is already active via `CLAUDE.md` — it loads automatically. Provide the Spec and the applicable Rule files directly.

**Other tools (Claude.ai, Cursor, ChatGPT):** Paste the content of `0-prompts/orchestrator.md` at the start of the conversation. Then provide the Spec and applicable Rule files (e.g., `@django-standards.md` + `@tailwind-global.md`).

### Step 3 — The Execution Loop

The Orchestrator runs the SDD cycle:

1. **Analysis** — reads the Spec, defines the Data Contract (the interface between layers).
2. **Parallel** — Tester Agent writes the test suite + Coder Agent writes the implementation simultaneously.
3. **Validation** — Validator Agent audits the Coder's output against the Spec and Rules. Returns `PASS` or `FAIL` with a violation list.
4. **Retry** — on `FAIL`, the Coder receives the violation list and retries. Max 2 retries; if still failing, the Orchestrator escalates to you.

### Step 4 — Close and Commit

Once the Validator returns `PASS`:

- Commit follows `2-rules/global/git-workflow.md` on the `develop` branch.
- Orchestrator updates `4-context/decisions.log` (if an architectural decision was made) and `4-context/project-map.md`.

---

## Memory Maintenance

When `decisions.log` reaches 10 entries, instruct the Orchestrator:

> "Execute Context Pruning: summarize architectural state into project-map.md and archive the old entries to 4-context/archive/."

---

## Tech Stack Quick Rules

| Stack | Key Constraint |
|-------|---------------|
| Django | Views are HTTP-only. All logic in `services.py`. Never call the ORM from a view. |
| NestJS | Zero `any`. DTOs with `class-validator`. Business logic lives in Services, not Controllers. |
| Next.js | `"use client"` only where strictly needed. Server Actions for mutations, not API routes. |
| All | Secrets via environment variables. Hardcoded credentials are a Validator violation. |
