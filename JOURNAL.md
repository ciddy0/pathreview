## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/119

**Issue title:** Add inline docstrings to all public methods in `core/services/`

**Tier:** [ ] Tier 1 [x] Tier 2 [ ] Tier 3

**Problem summary:**
The service layer in `core/services/` is entirely undocumented — none of its public methods carry docstrings, so a reader has to infer each method's parameters, return shape, and failure modes from the implementation. This matters more here than in most modules because the services sit between the API routes and the database models, so they are the layer a new contributor reads first when tracing a request. A successful fix adds Google-style docstrings (description, Args, Returns, Raises) to every public method in `profile_service.py` and `review_service.py`, accurately describing the exceptions each one actually raises rather than restating the method name. The issue also lists `core/services/notification_service.py`, but that file does not exist in the current tree — worth confirming on the issue before starting.

**Branch name:** `docs/119-core-services-docstrings`

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger
