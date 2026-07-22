# PLAN — Issue #119: Google-style docstrings for `core/services/`

## Problem

The service layer is the seam between the API routes and the database models, so it is the
first code a new contributor reads when tracing a request. Its public functions carry only a
one-line summary — no statement of what they accept, what they return, or how they fail.

The issue calls this "no docstrings," which is imprecise: every public function has a summary
line. The real gap is the structured **Args / Returns / Raises** sections.

## Reproduction

Introspect the two modules and report which public coroutines carry Google-style sections:

```bash
.venv/bin/python -c "
import inspect, core.services.profile_service as p, core.services.review_service as r
for mod in (p, r):
    print('='*60); print(mod.__name__)
    for name, fn in inspect.getmembers(mod, inspect.iscoroutinefunction):
        if name.startswith('_') or fn.__module__ != mod.__name__: continue
        d = inspect.getdoc(fn) or ''
        secs = [s for s in ('Args:','Returns:','Raises:') if s in d]
        print(f'  {name}: {len(d.splitlines())} line(s) | sections: {secs or \"NONE\"}')
"
```

Observed — **8 of 8 public functions report `sections: NONE`**:

| Module | Function | Docstring lines | Sections |
|---|---|---|---|
| `profile_service` | `create_profile` | 1 | NONE |
| `profile_service` | `get_profile` | 1 | NONE |
| `profile_service` | `update_profile` | 1 | NONE |
| `profile_service` | `delete_profile` | 2 | NONE |
| `review_service` | `create_review` | 1 | NONE |
| `review_service` | `get_review` | 1 | NONE |
| `review_service` | `list_reviews` | 2 | NONE |
| `review_service` | `process_review` | 9 | NONE |

`process_review` is the interesting case: 9 lines describing a numbered pipeline, but nothing
a caller can use to know what happens on failure.

## Why this is worth doing carefully

Error behavior is inconsistent across the layer and invisible from the signatures:

- `delete_profile` — catches, logs, calls `await db.rollback()`, then **re-raises**
  ([profile_service.py:110-113](core/services/profile_service.py#L110-L113)). Callers must
  handle exceptions.
- `process_review` — catches everything, marks the review `failed`, and returns `None`. Its
  recovery path is itself wrapped in a second `try/except` that swallows a failed status
  update ([review_service.py:182-194](core/services/review_service.py#L182-L194)). It
  **never propagates**, so `Raises:` should say so explicitly.
- `get_profile` / `get_review` / `update_profile` return `None` for both "not found" and
  "not owned by this user" — the ownership check is silent, which callers need told.

Documenting these accurately requires reading control flow, not transcribing signatures.

## Approach

1. Read each function end to end, including the four private helpers `process_review`
   delegates to, since they determine its observable failure behavior.
2. Write Google-style docstrings: summary line, blank line, `Args:`, `Returns:`, `Raises:`.
3. Omit `Raises:` where a function genuinely cannot raise, rather than padding it.
4. Describe the ownership-check semantics wherever `None` is overloaded.
5. Re-run the reproduction snippet — every public function should list all applicable
   sections.
6. Run `make check` (ruff, black, mypy) and `make test-unit`.

## Scope

**In:** the 8 public functions in `profile_service.py` and `review_service.py`.

**Out, and deliberately so:**

- The 4 private helpers in `review_service.py` (`_run_ingestion_pipeline`,
  `_run_agent_orchestration`, `_run_rag_retrieval_generation`, `_run_safety_checks`).
- The unannotated `db` parameter throughout — should be `AsyncSession`. Real, but a typing
  change does not belong in a docs PR.
- Implicit-optional defaults in `create_profile` (`resume_filename: str = None` should be
  `str | None`). Same reasoning; file separately.
- `core/services/notification_service.py`, listed in the issue but **absent from the tree**.

## Open questions

1. Was `notification_service.py` removed, renamed, or never landed? Asking on the issue
   before shipping two files out of three.
2. Should `Raises:` document exceptions from the SQLAlchemy layer (e.g. `IntegrityError` on
   a duplicate profile), or only exceptions this code raises deliberately? Affects
   `create_profile` and `create_review`.

## Risks

Low blast radius — docstrings only, no behavior change. The failure mode is *inaccuracy*:
a docstring claiming `process_review` raises on failure would be worse than the current
silence. Every `Raises:` section gets checked against the code path rather than assumed.
