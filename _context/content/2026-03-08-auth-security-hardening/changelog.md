<!-- [agent: content] Generated from SPEC-003 -->

## [0.6.0] - 2026-03-08

### Changed
- **Stricter login rate limiting**: Per-account login attempts reduced from 50 to 10 per 15 minutes to better resist brute-force attacks
- **CSRF validation tightened**: Only `same-origin` requests are now accepted (previously also allowed `same-site`, which includes sibling subdomains). Configured `CSRF_ALLOWED_ORIGINS` is now authoritative and no longer auto-includes the current request origin
- **Password reset token validation**: `validate-reset-token` endpoint changed from GET to POST — tokens are no longer sent in query strings, reducing exposure via browser history, server logs, and referrer headers

### Fixed
- **Password reset token lifecycle**: Requesting a new reset token now invalidates all previous active tokens for that user. A successful password reset also invalidates all remaining tokens. Previously, old tokens could remain valid alongside newer ones
- **Login timing leak**: Login requests for non-existent accounts now perform a dummy bcrypt verification, making response times consistent regardless of whether the email exists — preventing user enumeration via timing analysis
- **Rate limiting resilience**: Authentication endpoints now return 503 (Service Unavailable) when Redis is down, instead of silently falling back to weaker per-process in-memory rate limiting

### Security
- Based on a full OWASP ASVS audit of the authentication system
- 7 High/Medium severity issues addressed across backend and frontend
- 3 new security-focused test cases added
