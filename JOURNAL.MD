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

**PLAN.md link:** [PLAN.md](./PLAN.md) (root of this fork; will update with the GitHub blob link once pushed)

**Walkthrough video (recommended):** [link to your Loom video, ≤2 min — recommended, not graded]

**Blockers or open questions:**
Biggest open question is IP resolution behind a reverse proxy (`X-Forwarded-For` vs `request.client.host`) — no trusted-proxy config exists in this codebase yet, so I'm deferring that to a follow-up rather than trusting a spoofable header now. See Risks & Unknowns in PLAN.md for the full reasoning.


## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
Implemented the fix per `PLAN.md`: added `RateLimitMiddleware` in `api/middleware/rate_limit.py` and wired it into `api/main.py` ahead of `CORSMiddleware`, so a 429 still gets CORS headers. It checks every request against an IP-based bucket and, when a valid JWT is present, an additional per-user bucket, reusing the existing `RateLimiter` in `safety/rate_limiter.py`. The Week 8 reproduction test (`tests/integration/test_rate_limiting_reproduction.py`) now passes. Added `tests/unit/test_rate_limit_middleware.py` with 5 tests covering IP-only requests, IP+user requests, either layer over its limit, and invalid-token fallback to anonymous. Both changes are committed on `feat/#70-rate-limiting-addition` and pushed to origin.

Before committing, re-verified the issue's premise against the actual code: grepped the whole codebase for any existing caller of `RateLimiter`/`check_rate_limit` outside `safety/` and found none — no request, authenticated or not, was ever throttled before this fix. That's why the middleware adds both layers rather than only the per-IP layer the issue title asked for; adding IP-only would have left the (also broken) per-user case unfixed.

Ran `make check`/`make test-unit` before and after: baseline had 53 pre-existing unit test failures and ~182 pre-existing lint errors unrelated to this issue; after the change, still exactly 53 failures (plus 5 new passing tests) and ruff/black/mypy are clean on every file touched — verified against the actual `pre-commit` hook directly, since its isolated mypy environment (only `mypy` + `types-redis` installed, no `fastapi`/`starlette`) type-checks differently than running mypy inside the full project venv.

**Next steps:**
Open a draft PR against `pathreview`, request peer/mentor review in Slack, and fill out the PR template — calling out the corrected root-cause understanding in the description. Address feedback, then finalize for Check-in 2.

**Blockers:**
None currently.

---

### Check-in 2 (end of week)

**PR link:** [#751 — Feat/#70 rate limiting addition](https://github.com/ascherj/pathreview/pull/751)

**Branch:** `feat/#70-rate-limiting-addition`

**What was built:**
Added `RateLimitMiddleware` (`api/middleware/rate_limit.py`), wired into `api/main.py` ahead of `CORSMiddleware`. It checks every request against an IP-based bucket and, when a valid JWT is present, an additional per-user bucket, reusing the existing `RateLimiter` in `safety/rate_limiter.py` — closing the gap where no request, authenticated or not, was ever throttled.

**Tests added or updated:**
`tests/unit/test_rate_limit_middleware.py` (new) — 5 tests against an isolated FastAPI app with only `RateLimitMiddleware` wired, mocking `RateLimiter.check_rate_limit`: IP over limit → 429 without calling the route; IP allowed, no auth header → route runs; IP allowed but user over limit → 429; both allowed → route runs; invalid token → falls back to IP-only. Also confirmed `tests/integration/test_rate_limiting_reproduction.py` (added Week 8) now passes.

**Self-review confirmation:** [x] make check passes [x] make test-unit passes
(Both pass in the sense the assignment defines for a codebase with documented pre-existing issues: `main` already has 53 pre-existing unit test failures and ~182 pre-existing lint errors unrelated to this issue, in files this PR doesn't touch. Verified before and after — counts are identical; this PR adds 5 new passing tests and zero new lint/type errors, confirmed against the actual `pre-commit` hook.)

**Draft PR feedback received from:** None yet — requested review in the class Slack channel, awaiting response.
