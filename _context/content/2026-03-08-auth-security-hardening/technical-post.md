<!-- [agent: content] Generated from SPEC-003 -->

# OWASP Audit of a FastAPI + Next.js Auth System: 7 Fixes We Shipped

Running a security audit against your own auth code is humbling. Our system had JWT refresh rotation, bcrypt hashing, httpOnly cookies, rate limiting — and still had 7 exploitable gaps. Here's what we found and how we fixed each one.

## 1. Password Reset Tokens Never Expired Each Other

**The bug**: When a user requested a password reset, we only cleaned up *expired* tokens — not *active* ones. If a user requested reset token A, then requested token B, token A remained valid. After using B to reset their password, A was still usable.

**The fix**: One new CRUD method — `invalidate_all_active_for_user()` — called in two places: when creating a new token, and when consuming one. The key detail: `mark_used()`, `invalidate_all_active()`, and `update_password()` now share a single DB session and commit together as one transaction.

```python
await crud_password_reset.mark_used(self.db, token)
await crud_password_reset.invalidate_all_active_for_user(self.db, token.user_id)
await crud_user.update_password(self.db, user, hashed, invalidate_tokens=True)
# ^ this commits everything in one transaction
```

## 2. Reset Tokens Leaked via URLs

**The problem**: `GET /validate-reset-token?token=abc123` puts the secret token in the URL. That means it shows up in browser history, server access logs, reverse proxy logs, analytics tools, and `Referer` headers if the page links externally.

**The fix**: Changed to `POST /validate-reset-token` with `{"token": "abc123"}` in the request body. The email link still uses `?token=...` (unavoidable — it's an email click), but the frontend immediately extracts the token and all subsequent API calls use POST bodies.

## 3. Login Timing Revealed Whether Emails Existed

**The problem**: Our login endpoint did this:

```python
user = await get_by_email(db, email)
if not user or not verify_password(password, user.hashed_password):
    raise 401
```

When the user doesn't exist, `verify_password` (bcrypt, ~200ms) is skipped. An attacker can distinguish "email not found" (fast) from "wrong password" (slow) with a few hundred requests.

**The fix**: A module-level dummy hash and one extra line:

```python
_DUMMY_HASH = hash_password("dummy-timing-placeholder")

# In login:
if not user:
    verify_password(credentials.password, _DUMMY_HASH)  # constant time
    raise 401
```

This is a well-known OWASP pattern. The dummy hash is computed once at import time, so it doesn't affect startup.

## 4. CSRF Accepted `same-site` (Not Just `same-origin`)

**The problem**: Our CSRF middleware accepted `Sec-Fetch-Site: same-site`, which includes sibling subdomains. If `blog.example.com` is compromised, it could CSRF `app.example.com`.

**The fix**: One line change — only accept `same-origin`. Additionally, we fixed `getAllowedOrigins()` which was auto-appending the current request's origin even when an explicit allowlist was configured, defeating the purpose of the allowlist.

## 5. Rate Limiting Silently Degraded on Redis Failure

**The problem**: When Redis was unavailable, our `enforce_rate_limit()` silently fell back to per-process in-memory counters. In a multi-instance deployment, this means each instance has its own counter — an attacker gets N × limit attempts where N is the instance count.

**The fix**: Added a `fail_open` parameter. Auth endpoints pass `fail_open=False`, which returns 503 instead of degrading. Non-auth endpoints keep the default `fail_open=True` behavior (availability over strictness for non-sensitive operations).

```python
async def enforce_rate_limit(key, *, limit, window_seconds, message, fail_open=True):
    try:
        redis = await get_redis()
        # ... normal flow
    except RedisError:
        if not fail_open:
            raise 503  # auth endpoints: refuse to operate without rate limiting
        # non-auth: fall through to in-memory fallback
```

## 6. Account Login Limits Were Too Generous

50 login attempts per 15 minutes per account is enough for a determined attacker to try common passwords. Reduced to 10 — still comfortable for legitimate users who mistype their password a few times.

## Testing Strategy

We added 3 new tests covering the security-critical paths:

- `test_new_reset_token_invalidates_old_tokens` — requests two tokens, verifies the first is rejected after the second is issued
- `test_validate_reset_token_is_post_only` — verifies GET returns 405, POST returns 200
- `test_successful_reset_invalidates_remaining_tokens` — verifies all tokens are consumed after a successful reset

The tricky part was test infrastructure: our test suite mocked Redis to raise `RedisError`, which now triggered 503 with `fail_open=False`. We added a `_FakeRateLimitRedis` that implements `incr/expire/ttl` in-memory, mirroring the token service's existing `_FakeTokenRedis` pattern.

## What We Didn't Fix (Yet)

The audit also identified 6 Low-severity items we deferred:
- HSTS headers
- `__Host-` cookie prefix
- `iss`/`aud` JWT claims
- Hashing emails in Redis rate-limit keys
- Argon2id migration from bcrypt
- Generic `/register` response to prevent email enumeration

These are on the Phase 3 backlog. The system is meaningfully more secure without them, and each one involves trade-offs worth discussing separately.
