# Domain Layer

This directory defines the **business concepts** of the project — pure, technology-agnostic. No Django models, no TypeScript interfaces, no database concerns. Just what the business IS and what it DOES.

## Why this layer exists

Specs in `3-specs/` describe what to BUILD. The domain layer describes what the BUSINESS MEANS. A spec says "create a registration endpoint." The domain says "a User has an email, a password, and can only register once per email."

Without this layer, business rules end up scattered across models, views, and serializers — each module defining its own interpretation of "User" or "Order."

## Flow

```
1-domain/          →      3-specs/          →       code
(business concepts)   (implementation plan)   (2-rules + 0-prompts)

Define entities        Write a spec that          Orchestrator
and their rules        references domain          generates code
                       entities by name
```

## What goes here

| File | Purpose |
|------|---------|
| `templates/entity-template.md` | Copy this to define a new domain entity |
| `[entity-name].md` | One file per core domain entity |

## Rules

- **No technology.** No "this maps to a Django model" or "this is a TypeScript interface." That is a Spec/Rules concern.
- **One file per entity.** `user.md`, `order.md`, `product.md`.
- **Reference, don't duplicate.** Specs reference entities from here by name. If the domain definition changes, update it here and regenerate the affected specs.
