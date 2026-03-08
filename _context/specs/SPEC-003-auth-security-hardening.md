---

## id: SPEC-003
title: Auth Security Hardening (OWASP Audit)
status: released
priority: P0
effort: M
created: 2026-03-07
author: agent:brainstorm

# SPEC-003: Auth Security Hardening (OWASP Audit)

## Problem

Full auth audit revealed 7 High/Medium issues and 6 Low issues deviating from OWASP ASVS standards. The most critical gaps are in password reset token lifecycle, login timing leaks, CSRF validation strictness, and rate limiting configuration. These must be addressed before production hardening.

## Proposed Solution

Three-phase fix plan targeting all High/Medium issues, with Low items as optional follow-ups.

---

## Phase 1 — Password Reset & Enumeration Fixes (High Priority)

### 1.1 Password Reset Token Lifecycle (High)

**Problem:** Multiple active reset tokens can coexist for one user. After a successful reset, older tokens remain valid.

**Fix (Backend):**

- `PasswordResetService.create_reset_token()`: Invalidate ALL active tokens for the user before creating a new one (not just expired ones).
- `PasswordResetService.reset_password()`: Invalidate ALL active tokens for the user after successful reset (not just the consumed one).
- Wrap password update + token invalidation in a single DB transaction.

**Files:**

- `src/app/services/password_reset_service.py` — add `invalidate_all_active_for_user()` calls
- `src/app/crud/password_reset.py` — add `invalidate_all_active_for_user(db, user_id)` CRUD method

### 1.2 Reset Token in URL/GET (Medium)

**Problem:** `?token=...` leaks via browser history, logs, referrer headers.

**Fix (Backend + Frontend):**

- **Backend:** Change `GET /validate-reset-token?token=...` → `POST /validate-reset-token` with token in request body.
- **Frontend:** Password reset link keeps `?token=...` for initial page load (unavoidable for email links), but the reset-password page immediately extracts and clears it from URL. All subsequent API calls send token in POST body only.

**Files:**

- `src/app/api/v1/endpoints/auth.py` — change `validate-reset-token` to POST
- `src/app/schemas/user.py` — add `ValidateResetTokenRequest` schema
- `scopelytics-ai-frontend/lib/server/auth.ts` — update `validateResetTokenServer()` to POST
- `scopelytics-ai-frontend/app/(auth)/reset-password/page.tsx` — extract token from URL, clear URL, send via POST

### 1.3 Login Timing Leak (Medium)

**Problem:** Non-existent user skips bcrypt → measurably faster response → user enumeration.

**Fix (Backend):**

- Add a module-level dummy hash constant in `auth.py`.
- When user lookup returns `None`, verify against the dummy hash before returning 401.

**Files:**

- `src/app/api/v1/endpoints/auth.py` — add dummy verify in login endpoint

---

## Phase 2 — CSRF & Rate Limiting (Medium Priority)

### 2.1 CSRF: Restrict `same-site` to `same-origin` (Medium)

**Problem:** `Sec-Fetch-Site: same-site` allows sibling subdomains to bypass CSRF.

**Fix (Frontend):**

- In `validateCsrfRequest()`, only accept `same-origin` (not `same-site`).

**Files:**

- `scopelytics-ai-frontend/lib/server/csrf.ts` — remove `same-site` from accepted values

### 2.2 CSRF: Stop auto-trusting requestOrigin (Medium)

**Problem:** `getAllowedOrigins()` always adds current `requestOrigin` even when an explicit allowlist is configured.

**Fix (Frontend):**

- When `CSRF_ALLOWED_ORIGINS` is set, use it as-is without appending `requestOrigin`.
- Only fall back to `requestOrigin` when no env var is configured.

**Files:**

- `scopelytics-ai-frontend/lib/server/csrf.ts` — fix `getAllowedOrigins()`

### 2.3 Tighten Login Rate Limits (Medium)

**Problem:** 50 req/15min per account is too generous for brute-force protection.

**Fix (Backend):**

- Reduce `LOGIN_ACCOUNT_REQUESTS_PER_15_MINUTES` default from 50 → 10.

**Files:**

- `src/app/core/config.py` — change default value

### 2.4 Rate Limit Redis Fallback: Fail Closed for Auth (Medium)

**Problem:** Redis outage silently degrades to per-process memory, weakening auth protection.

**Fix (Backend):**

- Add a `fail_open` parameter to `enforce_rate_limit()`.
- Auth endpoints (`login`, `register`, `forgot-password`) pass `fail_open=False` — raises 503 on Redis failure.
- Non-auth endpoints keep current fail-open behavior.

**Files:**

- `src/app/core/business_rate_limit.py` — add `fail_open` param
- `src/app/api/v1/endpoints/auth.py` — pass `fail_open=False`

---

## Phase 3 — Hardening (Low Priority, Optional)


| #   | Item                         | File(s)                                 |
| --- | ---------------------------- | --------------------------------------- |
| 3.1 | Add HSTS header              | `security_headers.py`, `next.config.ts` |
| 3.2 | Cookie `__Host-` prefix      | `lib/server/auth.ts`                    |
| 3.3 | Add `iss`/`aud` JWT claims   | `security.py`, `dependencies.py`        |
| 3.4 | Hash emails in Redis keys    | `auth.py` rate limit keys               |
| 3.5 | Upgrade to Argon2id          | `security.py`, `requirements.txt`       |
| 3.6 | Generic `/register` response | `auth.py`                               |


---

## Acceptance Criteria

### Phase 1

- Creating a new reset token invalidates all existing active tokens for that user
- Successful password reset invalidates all remaining active tokens for that user
- Token invalidation + password update happen in one DB transaction
- `validate-reset-token` is POST (not GET), token in body
- Frontend clears token from URL after extraction
- Login response time is constant regardless of user existence
- All existing auth tests pass
- New tests cover: multi-token invalidation, POST validation, timing equivalence

### Phase 2

- CSRF rejects `Sec-Fetch-Site: same-site` (only accepts `same-origin`)
- Configured `CSRF_ALLOWED_ORIGINS` is authoritative (no auto-append)
- Login rate limit per account ≤ 10/15min
- Auth rate limiting returns 503 when Redis is unavailable
- Existing CSRF and rate limit tests pass

### Phase 3 (optional)

- HSTS header present in production responses
- Cookies use `__Host-` prefix
- JWT contains `iss` and `aud` claims

## Open Questions

- Should `/register` return a generic response to prevent email enumeration? (Product decision — may hurt UX)
- Is immediate access token revocation (server-side session table) needed for MVP? (Deferred to Phase 4 if needed)

