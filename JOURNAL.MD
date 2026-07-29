## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/70

**Issue title:** Add rate limiting per IP address in addition to per user

**Tier:** [ ] Tier 1  [X] Tier 2  [ ] Tier 3

**Problem summary:**
Currently, unauthenticated requests are not rate-limited at all. As it is, the current rate-limiter
found in safety/rate_limiter.py limits requests per authenticated user-id. Adding a rate limiter per IP-address as a secondary layer would cover the issue with unauthenticated requests. 
[In 3–5 sentences, in your own words: what the issue is (not a copy-paste of
the title), what is currently broken or missing, and what a successful fix
would accomplish. Naming the part of the codebase it affects is helpful context.]

**Branch name:** feat/#70-rate-limiting-addition

**Setup confirmation:** [X] App runs locally at localhost:5173

**Cohort ledger:** [X] Issue added to cohort ledger


## Week 8 — Reproduction & solution planning

**Reproduction commit link:** (fill in after committing — will be the commit adding `tests/integration/test_rate_limiting_reproduction.py`)

**Reproduction summary:**
Confirmed `RateLimiter` in `safety/rate_limiter.py` is never wired into the app. No route or middleware calls `check_rate_limit`, so no request is throttled today. Reproduced two ways: (1) manually, ran the app locally against Dockerized Postgres/Redis and fired 150 rapid unauthenticated `curl` requests at `/`, all 150 returned `200` with zero `429`s; (2) added a failing integration test, `tests/integration/test_rate_limiting_reproduction.py`, which sends `rate_limit_per_minute + 20` unauthenticated requests through a `TestClient` and asserts a `429` appears — it currently fails, proving the gap exists at the point where `api/main.py` builds the middleware stack (no `RateLimitMiddleware` is registered, unlike `RequestIDMiddleware`).

**PLAN.md link:** [link to PLAN.md in your fork]

**Walkthrough video (recommended):** [link to your Loom video, ≤2 min — recommended, not graded]

**Blockers or open questions:**
[Anything you're still uncertain about going into Week 9, or leave blank]

