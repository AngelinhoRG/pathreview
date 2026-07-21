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
