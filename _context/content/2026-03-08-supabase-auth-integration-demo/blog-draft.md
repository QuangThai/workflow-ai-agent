[agent: content]

# A Practical Supabase Auth Foundation for NestJS and Next.js 16

Authentication often fails at the boundaries: frontend sessions look valid while backend APIs reject tokens, or route guards pass while sensitive data remains exposed. This update defines a unified foundation to prevent exactly those issues.

## The Problem
Modern full-stack apps frequently combine SSR frontend behavior with API-first backend services. Without a clear auth boundary, teams encounter session mismatches, redirect loops, and inconsistent authorization checks. These issues become more visible in production where cookie and token refresh behavior is stricter.

## The Solution
We designed a hybrid Supabase auth model:
- Next.js 16 manages sign-in, callback, logout, and SSR session lifecycle using `@supabase/ssr`.
- NestJS validates Supabase bearer JWTs using JWKS and enforces API route protection.
- Authorization follows defense-in-depth: UX checks in frontend, security-critical checks in backend/data layer.

## How It Works
1. Frontend uses separate browser and server Supabase clients.
2. Proxy/middleware refreshes session tokens and synchronizes cookies on request/response boundaries.
3. Backend validates token signature and claims before protected API logic executes.
4. Security hardening adds cookie/session policy consistency, auth logging, and operational troubleshooting guidance.

## Why This Matters
This foundation is built for reliability and scale:
- Fewer SSR auth edge-case failures.
- Clear ownership of authentication vs authorization responsibilities.
- Better readiness for OAuth expansion, RLS adoption, and key rotation.

## Success Metrics
- Redirect loop rate on auth navigation:
  - Baseline: Not measured
  - Target: 0 redirect loops in smoke tests
  - Current status: Pending implementation
- Protected API token enforcement:
  - Baseline: Not measured
  - Target: 100% expected `401`/`200` behavior in integration tests
  - Current status: Pending implementation
- Session refresh reliability:
  - Baseline: Not measured
  - Target: >=99% successful refresh behavior in staged verification
  - Current status: Pending implementation

## Try It Out
Follow the phased implementation plan in `SPEC-002`:
- Phase 1: Next.js SSR auth foundation
- Phase 2: NestJS JWT enforcement
- Phase 3: Security hardening and operational readiness
