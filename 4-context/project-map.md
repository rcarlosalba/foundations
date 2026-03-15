# Project Map

> **Maintained by:** Orchestrator Agent (auto-updated after each validated cycle)
> **Purpose:** Single-page snapshot of the project's current architectural state. Not a spec — a live map.

---

## Project Metadata

| Field | Value |
|-------|-------|
| **Project Name** | [Project Name] |
| **Version** | 0.1.0 |
| **Last Updated** | YYYY-MM-DD |
| **Active Branch** | develop |

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | [Django / NestJS] |
| Frontend | [Next.js / Alpine+Tailwind+Django templates] |
| Database | [PostgreSQL / SQLite] |
| Auth | [Session / JWT] |
| Deployment | [Vercel / Railway / Docker] |

---

## Module Registry

> List every implemented module. Status: `planned` | `in-progress` | `done`

| Module | Status | Key Files | Notes |
|--------|--------|-----------|-------|
| _(example) User Auth_ | `done` | `apps/users/services.py`, `apps/users/models.py` | JWT-based, no social login |
| _[Module Name]_ | `planned` | — | — |

---

## Active Data Contracts

> The live interfaces between system layers. Becomes stale when a module is refactored — update immediately.

```
[Example]
POST /api/users/register
Request:  { email: string, password: string, full_name: string }
Response: { id: number, email: string, created_at: string }
Errors:   400 { field_errors: Record<string, string[]> }
          409 { detail: "Email already registered" }
```

---

## Architectural Decisions (Summary)

> Brief index. Full entries are in `decisions.log`.

| ADR | Title | Impact |
|-----|-------|--------|
| ADR-001 | _(example) Service Layer for all business logic_ | No ORM calls in views ever |
| — | — | — |

---

## Known Constraints & Out-of-Scope Items

> Explicit boundaries. Prevents gold-plating and "but what about X?" scope creep.

- [Constraint or out-of-scope item]
- [Constraint or out-of-scope item]
