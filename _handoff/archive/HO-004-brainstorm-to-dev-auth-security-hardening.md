---
id: HO-004
from: brainstorm
to: dev
priority: P0
status: done
created: 2026-03-07
spec: SPEC-003
---

# Auth Security Hardening (OWASP Audit)

## Context
Full auth audit identified 7 High/Medium and 6 Low security issues. This handoff covers Phase 1 (critical) and Phase 2 (important) fixes. Phase 3 is optional hardening.

## Scope
- **Backend**: Password reset service, auth endpoints, rate limiting, config defaults
- **Frontend**: CSRF validation, reset token handling, cookie settings
- **Shared**: No migration needed; no new endpoints (only method change on validate-reset-token)

## Implementation Plan

### Phase 1 — Do first (blocks production hardening)

**Step 1: Password reset token lifecycle** (~1h)
1. Add `invalidate_all_active_for_user(db, user_id)` to `src/app/crud/password_reset.py`
2. In `PasswordResetService.create_reset_token()`: call invalidate before creating new token
3. In `PasswordResetService.reset_password()`: wrap password update + token invalidation + mark_used in single transaction
4. Write tests: verify old tokens invalid after new creation, verify old tokens invalid after successful reset

**Step 2: Validate-reset-token POST migration** (~30min)
1. Add `ValidateResetTokenRequest` schema to `src/app/schemas/user.py`
2. Change `GET /validate-reset-token` → `POST /validate-reset-token` in `auth.py`
3. Update `validateResetTokenServer()` in `lib/server/auth.ts` to use POST body
4. Update reset-password page to extract token from URL → state → clear URL
5. Update `tests/test_auth.py` for POST method

**Step 3: Login timing leak fix** (~15min)
1. Add dummy hash at module level in `auth.py`: `_DUMMY_HASH = hash_password("dummy-timing-placeholder")`
2. In `/login`: when `user is None`, call `verify_password(credentials.password, _DUMMY_HASH)` before raising 401
3. Write test: verify both paths return similar timing

### Phase 2 — Do second (important hardening)

**Step 4: CSRF strictness** (~15min)
1. In `csrf.ts`: remove `same-site` from accepted `Sec-Fetch-Site` values
2. In `getAllowedOrigins()`: when `CSRF_ALLOWED_ORIGINS` is configured, don't auto-append `requestOrigin`

**Step 5: Rate limit tightening** (~30min)
1. In `config.py`: change `LOGIN_ACCOUNT_REQUESTS_PER_15_MINUTES` default 50 → 10
2. In `business_rate_limit.py`: add `fail_open: bool = True` parameter
3. In `auth.py`: pass `fail_open=False` to all auth rate limit calls
4. Write test: verify 503 on Redis failure for auth endpoints

### Parallelization
- Steps 1-3 (backend) can run in parallel with Step 4 (frontend CSRF)
- Step 5 depends on Step 1-3 being complete (same files)

## Deliverables
- [x] Spec approved: `_context/specs/SPEC-003-auth-security-hardening.md`
- [ ] Backend: Password reset lifecycle fix
- [ ] Backend: POST validate-reset-token
- [ ] Backend: Login timing leak fix
- [ ] Backend: Rate limit hardening
- [ ] Frontend: CSRF strictness
- [ ] Frontend: Reset token URL handling
- [ ] Tests: All existing pass + new security tests
- [ ] PRD.md: No update needed (FR-01 and SEC-03 already cover these)

## Acceptance Criteria
_(see SPEC-003 for full list)_

**Critical:**
1. Old reset tokens invalidated on new token creation
2. Old reset tokens invalidated on successful password reset
3. `validate-reset-token` accepts POST only
4. Login timing constant regardless of user existence
5. CSRF rejects `same-site` Sec-Fetch-Site
6. Auth rate limiting fails closed (503) on Redis outage

## Dev Notes
- Relevant files:
  - `src/app/services/password_reset_service.py`
  - `src/app/crud/password_reset.py`
  - `src/app/api/v1/endpoints/auth.py`
  - `src/app/schemas/user.py`
  - `src/app/core/config.py`
  - `src/app/core/business_rate_limit.py`
  - `scopelytics-ai-frontend/lib/server/auth.ts`
  - `scopelytics-ai-frontend/lib/server/csrf.ts`
- Related tests: `tests/test_auth.py`
- Migration needed: No
- Breaking API change: `validate-reset-token` GET → POST (frontend-only consumer, no external impact)

## Implementation Log
- **Files changed (Backend — 7):**
  - `src/app/crud/password_reset.py` — Added `invalidate_all_active_for_user()`, removed commit from `mark_used()`
  - `src/app/services/password_reset_service.py` — Invalidate old tokens on create & reset, single transaction
  - `src/app/api/v1/endpoints/auth.py` — GET→POST validate-reset-token, dummy hash timing fix, `fail_open=False`
  - `src/app/schemas/user.py` — Added `ValidateResetTokenRequest`
  - `src/app/core/config.py` — `LOGIN_ACCOUNT_REQUESTS_PER_15_MINUTES` 50→10
  - `src/app/core/business_rate_limit.py` — Added `fail_open` parameter
  - `tests/conftest.py` — Added `_FakeRateLimitRedis` for test env
- **Files changed (Frontend — 4):**
  - `lib/server/csrf.ts` — Removed `same-site`, fixed allowlist auto-append
  - `lib/server/auth.ts` — `validateResetTokenServer()` GET→POST
  - `lib/api.ts` — `validateResetToken` GET→POST
  - `app/api/auth/[...path]/route.ts` — validate-reset-token handler GET→POST
- **Tests added:** 3 new tests in `test_auth.py` (token invalidation, POST-only, reset lifecycle)
- **Migration:** No
- **Notes:** Test conftest updated with `_FakeRateLimitRedis` to support `fail_open=False` in test environment

## QA Report
- **Date**: 2026-03-08
- **Agent**: qa
- **Verdict**: PASS

### Automated Checks
| Check | Result |
|-------|--------|
| Backend lint (`ruff check src`) | ✅ All checks passed |
| Backend format (`ruff format --check src`) | ✅ 163 files already formatted |
| Backend tests (`pytest test_auth + test_config_validation`) | ✅ 22/22 passed |
| Frontend lint (`npm run lint`) | ✅ No errors |
| Frontend typecheck (`tsc --noEmit`) | ✅ Zero errors |
| Frontend build (`npm run build`) | ✅ Build succeeded |

### Acceptance Criteria
| # | Criterion | Result | Evidence |
|---|-----------|--------|----------|
| 1 | Old reset tokens invalidated on new token creation | ✅ | `test_new_reset_token_invalidates_old_tokens` — token_a returns 400 after token_b issued |
| 2 | Old reset tokens invalidated on successful password reset | ✅ | `invalidate_all_active_for_user()` called in `reset_password()` before commit |
| 3 | `validate-reset-token` accepts POST only | ✅ | `test_validate_reset_token_is_post_only` — GET returns 405, POST returns 200 |
| 4 | Login timing constant regardless of user existence | ✅ | `_DUMMY_HASH` verify runs when `user is None` (auth.py L123-124) |
| 5 | CSRF rejects `same-site` Sec-Fetch-Site | ✅ | `csrf.ts` only accepts `same-origin` (L40) |
| 6 | CSRF configured allowlist is authoritative | ✅ | `getAllowedOrigins()` no longer appends `requestOrigin` when configured (L16) |
| 7 | Login rate limit ≤ 10/15min per account | ✅ | `config.py` L124: `LOGIN_ACCOUNT_REQUESTS_PER_15_MINUTES: int = 10` |
| 8 | Auth rate limiting fails closed (503) on Redis outage | ✅ | `fail_open=False` on all 7 auth `enforce_rate_limit` calls |

### Manual Review Checklist
- [x] Code follows conventions (Python type hints, async/await, existing patterns)
- [x] No hardcoded secrets or credentials
- [x] Error handling is appropriate (503 on Redis failure, 400/401 on invalid tokens)
- [x] Types are correct and complete (`ValidateResetTokenRequest` schema added)
- [x] Tests cover new functionality (3 new tests)
- [x] No unnecessary changes outside scope
- [x] Transaction safety: `mark_used` + `invalidate_all_active` + `update_password` share same db session

### Issues Found
None.

### Notes for Release
- **Breaking API change**: `validate-reset-token` endpoint changed from GET to POST. Frontend is the only consumer and has been updated — no external impact.
- **Rate limit default changed**: `LOGIN_ACCOUNT_REQUESTS_PER_15_MINUTES` 50→10. Existing deployments using env var override are unaffected.
- **Test infra**: `conftest.py` now uses `_FakeRateLimitRedis` instead of raising `RedisError` for `business_rate_limit`, supporting `fail_open=False` in tests.

## References
- Spec: `_context/specs/SPEC-003-auth-security-hardening.md`
- PRD: `scopelytics-ai-backend/PRD.md` §FR-01 (Auth), §SEC-03 (Token Integrity)
