# Bug Report: [Short Title]

## 1. Observed vs Expected Behavior
- **Observed:** What is actually happening?
- **Expected:** What should happen instead?

## 2. Location
- **Page / Component / Endpoint:** (e.g., `/checkout`, `<DarkModeToggle>`, `POST /api/orders`)
- **Environment:** (e.g., browser, OS, local/staging/production)

## 3. Steps to Reproduce
1. Go to...
2. Click / trigger...
3. See error...

## 4. Error Output
> Paste any console errors, stack traces, or logs here. If none, write "No errors visible."

```
[error output here]
```

## 5. Severity
- [ ] **Blocking** — Core functionality is broken, no workaround
- [ ] **High** — Major feature broken, workaround exists
- [ ] **Low** — Cosmetic / intermittent issue

## 6. Suspected Files
> Best guess at where the root cause lives. Orchestrator will verify.
- `path/to/file.ts`
- `path/to/other-file.py`

## 7. Regression Test Requirement
**Mandatory.** The Coder Agent must write a test that:
- Reproduces the exact failure scenario described above.
- Passes after the fix is applied.
- Lives alongside existing tests for the affected module.
