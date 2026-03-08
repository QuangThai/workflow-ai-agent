---
id: HO-005
from: dev
to: release
priority: P0
status: pending
created: 2026-03-08
spec: SPEC-003
---

# Release: Auth Security Hardening (OWASP Audit)

## Summary
Security hardening of the authentication system based on a full OWASP audit. Fixes 7 High/Medium issues: password reset token lifecycle vulnerabilities, login timing leak enabling user enumeration, CSRF validation too permissive, and rate limiting gaps on Redis failure. No new features — purely security fixes.

## Changes

### Backend
- `src/app/crud/password_reset.py` — Added `invalidate_all_active_for_user()`, `mark_used()` no longer auto-commits
- `src/app/services/password_reset_service.py` — Invalidate all active tokens on new token creation and on successful reset; single-transaction password update
- `src/app/api/v1/endpoints/auth.py` — `validate-reset-token` GET→POST, dummy hash for login timing protection, `fail_open=False` on all auth rate limits
- `src/app/schemas/user.py` — Added `ValidateResetTokenRequest` schema
- `src/app/core/config.py` — `LOGIN_ACCOUNT_REQUESTS_PER_15_MINUTES` default 50→10
- `src/app/core/business_rate_limit.py` — Added `fail_open` parameter (default `True`, auth uses `False`)
- `tests/conftest.py` — `_FakeRateLimitRedis` for test env support
- `tests/test_auth.py` — 3 new security tests

### Frontend
- `lib/server/csrf.ts` — Only accept `same-origin` (removed `same-site`); configured allowlist is authoritative
- `lib/server/auth.ts` — `validateResetTokenServer()` uses POST body
- `lib/api.ts` — `validateResetToken` GET→POST
- `app/api/auth/[...path]/route.ts` — `validate-reset-token` handler GET→POST, added to `POST_ONLY_ACTIONS`

### Database
- Migration: No
- No schema changes

## QA Status
- All automated checks: ✅
- Acceptance criteria: ✅ 8/8
- QA report: `_handoff/archive/HO-004-brainstorm-to-dev-auth-security-hardening.md`

## Deploy Notes
- **Environment variables**: No new env vars required
- **Breaking changes**: `validate-reset-token` endpoint changed from GET to POST — frontend is the only consumer and is updated in this release
- **Rate limit default**: `LOGIN_ACCOUNT_REQUESTS_PER_15_MINUTES` changed 50→10. Deployments with explicit env override are unaffected
- **Redis dependency**: Auth endpoints now return 503 if Redis is unavailable (previously fell through to in-memory fallback). Ensure Redis is healthy before deploy.
- **Rollback plan**: Revert all 12 files to previous version. No DB migration to undo. Rate limit change is config-only (can override via env var).

## Post-Deploy Verification
- [ ] Health check passes (`/health`, `/ready`)
- [ ] Smoke test: Login with valid credentials → 200
- [ ] Smoke test: Login with nonexistent email → 401 (not 500, timing similar to valid user)
- [ ] Smoke test: `POST /validate-reset-token` with body `{"token":"invalid"}` → 200 `{"valid":false}`
- [ ] Smoke test: `GET /validate-reset-token?token=x` → 405
- [ ] Monitor: Redis connectivity; auth 503 error rate; login latency distribution
- [ ] Monitor: Password reset success rate (should be unchanged for legitimate users)
