## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/119

**Issue title:** Add inline docstrings to all public methods in `core/services/`

**Tier:** [ ] Tier 1 [x] Tier 2 [ ] Tier 3

**Problem summary:**
The service layer sits between the API routes and the database models, so it is the code a
new contributor reads first when tracing a request — but its functions document themselves
only with a one-line summary, with no statement of what they take, what they hand back, or
how they fail. The issue describes this as having "no docstrings," which is slightly off:
all eight public functions do have a short summary line, so the actual gap is the structured
Args / Returns / Raises sections, not the docstrings themselves. That gap matters most
around error behavior, which is genuinely inconsistent across the layer and invisible from
the signatures: `delete_profile` rolls back and re-raises on failure, while `process_review`
swallows its exceptions, marks the review `failed`, and returns `None` — a caller cannot
tell those apart without reading both bodies. A successful fix gives every public function
in `profile_service.py` and `review_service.py` a Google-style docstring whose Raises
section reflects the control flow that is actually there, so callers can reason about
failure without opening the implementation. One thing to confirm on the issue first: it
lists `core/services/notification_service.py`, which does not exist in the current tree.

**Branch name:** `docs/119-core-services-docstrings`

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger
