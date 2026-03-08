[agent: brainstorm]

# Research: Supabase Auth with NestJS + Next.js 16

Date: 2026-03-08
Topic: Implement Auth foundation using Supabase for Next.js 16 frontend and NestJS backend.

## Fact Ledger

### Given / Verified Facts
- Workspace currently has no app code under `apps/`; this is discovery-first planning.
- `_context/product-state.md` indicates prior auth foundation work was released as `SPEC-001` (2026-03-08), but `_context/specs/` has no files.
- Next.js 16 App Router supports auth patterns via server actions/components + proxy/middleware checks.
- Supabase SSR guidance for Next.js recommends `@supabase/ssr` with separate browser/server clients and proxy middleware for session refresh.
- NestJS has official JWT guard/strategy patterns for API protection.

### Facts to Look Up
- Latest Supabase SSR guidance differences between `getSession`, `getUser`, and `getClaims`.
- Next.js 16 specific guidance for auth route protection and cookie handling.
- Security baselines for session cookies and JWT verification.

### Facts to Derive
- Best integration boundary between Next.js + NestJS when Supabase Auth is identity provider.
- Phase breakdown for shipping minimal secure auth foundation.

### Educated Guesses
- Best fit is: Next.js handles interactive auth/session, NestJS verifies bearer JWT via JWKS and performs authorization for API routes. (confidence: High)
- Need short-lived access tokens + refresh through Supabase-managed session cookies in frontend path. (confidence: High)

## Top Sources Consulted

Tier 1 (official/spec):
- Next.js Authentication Guide: https://nextjs.org/docs/app/guides/authentication
- Next.js Middleware/Proxy & cookies docs: https://nextjs.org/docs/app/api-reference/file-conventions/middleware
- Supabase Next.js SSR guide: https://supabase.com/docs/guides/auth/server-side/nextjs
- Supabase SSR advanced guide: https://supabase.com/docs/guides/auth/server-side/advanced-guide
- Supabase JWT signing keys: https://supabase.com/docs/guides/auth/signing-keys
- Supabase JWT guide: https://supabase.com/docs/guides/auth/jwts
- NestJS Authentication: https://docs.nestjs.com/security/authentication
- NestJS Passport/JWT recipe: https://docs.nestjs.com/recipes/passport
- OWASP Session Management Cheat Sheet: https://github.com/owasp/cheatsheetseries/blob/master/cheatsheets/Session_Management_Cheat_Sheet.md

Tier 2 (secondary, corroborated by Tier 1):
- Supabase troubleshooting for Next.js SSR auth: https://supabase.com/docs/guides/troubleshooting/how-do-you-troubleshoot-nextjs---supabase-auth-issues-riMCZV

## Distilled Best Practices (project-relevant)
- Use `@supabase/ssr` (not legacy auth helpers) for Next.js App Router SSR auth.
- Maintain 3 client contexts in Next.js: browser client, server client, proxy/middleware update session flow.
- In server-side authorization checks, do not trust unverified session payload from cookies.
- Prefer `getUser()` when you need authoritative server-side user validation; use `getClaims()` for local signature/claims validation workflows where appropriate.
- Keep auth cookies secure (`Secure`, `SameSite=Lax` baseline; production HTTPS mandatory).
- Scope proxy/middleware matcher to avoid static assets and keep overhead controlled.
- In NestJS, protect API endpoints using guard + JWT verification and attach validated claims/user to request context.
- Verify Supabase JWTs using JWKS and key rotation-friendly flow; avoid hardcoding shared secrets when asymmetric signing keys are available.
- Put authorization close to data access (RLS and backend checks), not only in middleware redirects.
- Plan for token refresh race/prefetch edge cases in Next.js; post-login landing page should avoid premature prefetched protected requests.

## Approaches Considered
- Rejected: Storing custom app auth state only in localStorage. Why: weak SSR alignment, higher XSS exposure, conflicts with Supabase SSR guidance.
- Rejected: Next.js-only route checks with no backend JWT verification. Why: leaves NestJS APIs insufficiently protected.
- Recommended: Hybrid boundary.
  - Next.js: session lifecycle + UX/auth routes.
  - NestJS: API authz/authn via bearer JWT validation and role checks.

## Recommendation with Trade-offs
- Recommendation: Implement Supabase Auth as central IdP with cookie-based SSR session in Next.js and JWT verification in NestJS.
- Pros: aligned with official docs, strong SSR UX, secure API boundary, supports key rotation and future OAuth/social providers.
- Cons: more moving parts (proxy + server/client clients + backend verifier), requires careful cookie/session troubleshooting in production.
- Confidence: High.

## Source Metadata Per Key Finding
- Finding: Next.js auth should use server-side cookie/session controls.
  - source_url: https://nextjs.org/docs/app/guides/authentication
  - publish_or_last_updated: unknown (page current as retrieved)
  - retrieved_at: 2026-03-08
  - confidence: High
- Finding: Supabase SSR for Next.js requires proxy/middleware token refresh flow and discourages trusting `getSession` for auth decisions.
  - source_url: https://supabase.com/docs/guides/auth/server-side/nextjs
  - publish_or_last_updated: unknown (page current as retrieved)
  - retrieved_at: 2026-03-08
  - confidence: High
- Finding: PKCE and cookie-shared session storage are core SSR flow assumptions.
  - source_url: https://supabase.com/docs/guides/auth/server-side/advanced-guide
  - publish_or_last_updated: 2026-02-20 (from indexed metadata)
  - retrieved_at: 2026-03-08
  - confidence: High
- Finding: Asymmetric JWT signing keys are preferred over legacy JWT secret.
  - source_url: https://supabase.com/docs/guides/auth/signing-keys
  - publish_or_last_updated: unknown (page current as retrieved)
  - retrieved_at: 2026-03-08
  - confidence: High
- Finding: NestJS JWT guards/strategy are standard for protected routes.
  - source_url: https://docs.nestjs.com/security/authentication
  - publish_or_last_updated: unknown (page current as retrieved)
  - retrieved_at: 2026-03-08
  - confidence: High
- Finding: Cookie/session security attributes and server-side invalidation are mandatory controls.
  - source_url: https://github.com/owasp/cheatsheetseries/blob/master/cheatsheets/Session_Management_Cheat_Sheet.md
  - publish_or_last_updated: rolling document
  - retrieved_at: 2026-03-08
  - confidence: High
