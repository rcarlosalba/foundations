---
description: Ruleset for Django application development. Focuses on 'The Django Way', strict separation of concerns via Service Layers, performance (ORM), and security.
globs: "**/*.py"
---

# Django Development Rules

**Core Goal:** Generate Django code that is idiomatic, secure, high-performance, and strictly maintainable.

## 1. Core Principles & Constraints
1. **Strict Minimalism:** Generate ONLY the code requested in the Spec. Do not add unsolicited features (pagination, search, extra fields) unless explicitly stated in the Spec.
2. **Type Hinting:** All functions, methods, and variables MUST use Python type hints (`->`, `:`, `List`, `Optional`, etc.). This is non-negotiable for system predictability.
3. **PEP 8:** Strict compliance. Classes in `PascalCase`, functions/variables in `snake_case`.

## 2. Architecture & Separation of Concerns
We strictly follow a **Service Layer Architecture** to keep Models and Views thin.

- **Views (`views.py` / `api.py`):** Are ONLY responsible for HTTP routing, extracting parameters, calling a Service, and returning an HTTP Response. 
    - ❌ **WRONG:** Complex `if/else` business logic or multiple ORM `create()` calls inside a view.
- **Services (`services.py`):** All business logic MUST reside here. Functions in `services.py` should take raw data or models as input and return processed data or raise custom exceptions.
- **Selectors (`selectors.py`):** Complex read-only ORM queries should reside here, returning standard querysets or dictionaries.

## 3. Models and ORM Performance
The Django ORM is the only authorized database interaction layer.

- **N+1 Anti-Pattern (CRITICAL):** NEVER perform database queries inside a loop. 
    - Use `select_related` for `ForeignKey`/`OneToOne`.
    - Use `prefetch_related` for `ManyToManyField`/Reverse `ForeignKey`.
- **Bulk Operations:** For creating or updating multiple records, ALWAYS use `bulk_create` or `bulk_update`. Do not iterate and `.save()`.
- **Atomic Transactions:** Any business operation in a Service that modifies more than one table MUST be wrapped in `@transaction.atomic`.

## 4. Security (Non-Negotiable)
1. **Secrets:** NEVER hardcode secrets. ALWAYS use `decouple.config()` to read from environment variables.
2. **Data Mutation:** Any form or API endpoint that mutates data (POST, PUT, DELETE) MUST be protected by CSRF (for templates) or proper Token/Session auth (for APIs).
3. **Input Validation:** ALWAYS use Django Forms or DRF Serializers/Pydantic schemas to validate incoming data. Never trust `request.POST.get()` directly for database inserts.

## 5. Directory Structure Reference
The AI must place new files in accordance with this structure:
```text
apps/
└── [app_name]/
    ├── models.py       # Data representation
    ├── views.py        # HTTP layer
    ├── services.py     # Business logic (CREATE/UPDATE/DELETE)
    ├── selectors.py    # Complex ORM queries (READ)
    ├── exceptions.py   # Domain-specific exceptions
    ├── urls.py
    └── tests/          # pytest-django based tests
```

---

### ✅ LLM Pre-Generation Checklist

Before finalizing any Django code, verify:

1. Is all business logic in `services.py`? Is `views.py` free of `if/else` and ORM calls?
2. Are there any N+1 queries — ORM calls inside a loop? Use `select_related` / `prefetch_related`.
3. Does any service function modify more than one table? Wrap it in `@transaction.atomic`.
4. Are all secrets read via `decouple.config()`, not hardcoded?
5. Is incoming data validated via a Django Form or DRF Serializer before any DB write?
6. Are all functions and variables fully type-hinted (`->`, `:`, `Optional`, `List`, etc.)?
7. Are custom exceptions defined in `exceptions.py`, not generic `Exception` raises?