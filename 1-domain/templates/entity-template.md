# Entity: [Entity Name]

> Copy this file to `1-domain/[entity-name].md` and fill in all sections.
> This is a business definition — no code, no framework, no database types.

---

## 1. Identity

| Field | Value |
|-------|-------|
| **Name** | [Entity Name] |
| **Description** | One sentence: what is this entity in business terms? |
| **Owner** | Which module/bounded context owns this entity? |

---

## 2. Properties

> Define each property in business terms. Use plain types: `text`, `number`, `boolean`, `date`, `enum`, `reference to [Entity]`.

| Property | Type | Required | Constraints | Description |
|----------|------|----------|-------------|-------------|
| `id` | number | yes | unique, auto-assigned | System identifier |
| `[property]` | [type] | yes/no | [max length, range, format, etc.] | [What it means] |

---

## 3. Invariants (Business Rules)

> Rules that MUST always be true for this entity to be valid. These become validation logic and test cases.

- **INV-01:** [e.g., "email must be unique across all Users"]
- **INV-02:** [e.g., "Order total must equal the sum of its line items"]
- **INV-03:** [Add as many as needed]

---

## 4. Lifecycle & States

> If this entity has states, define them. If it's stateless, remove this section.

```
[State A] ──── [trigger] ──→ [State B] ──── [trigger] ──→ [State C]
   │                                                            │
   └────────────────── [trigger] ──────────────────────────────┘
```

| State | Meaning | Allowed Transitions |
|-------|---------|---------------------|
| `[state_a]` | [Business meaning] | → `[state_b]` on [condition] |

---

## 5. Relationships

| Related Entity | Type | Description |
|----------------|------|-------------|
| `[Entity]` | one-to-many / many-to-many / belongs-to | [Why this relationship exists] |

---

## 6. Domain Events

> Events this entity emits when something significant happens. These inform async flows, notifications, and audit logs.

| Event | Trigger | Consumers |
|-------|---------|-----------|
| `[EntityName]Created` | When a new instance is first persisted | [Who cares about this?] |
| `[EntityName]StatusChanged` | When `status` transitions | [Who cares?] |

---

## 7. Out of Scope for this Entity

> Explicit list of what this entity does NOT own or handle. Prevents scope creep in specs.

- [This entity does not handle X — that belongs to [Other Entity]]
- [Billing/payments are out of scope here, see `order.md`]
