[agent: content]

# Building Supabase Auth Across Next.js 16 and NestJS: Boundary-First Design

## Context
The core challenge was not choosing an auth provider, but defining a reliable boundary between SSR session behavior and API authorization. In this architecture, frontend and backend are both security participants, but with different responsibilities.

## Approach
We selected Supabase as the identity provider and split responsibilities by execution context:
- Next.js 16 (App Router): session lifecycle and authentication UX.
- NestJS API: token verification and authorization enforcement.

This avoids a common anti-pattern where frontend route checks are treated as sufficient security controls for backend resources.

## What We Considered and Rejected
- Frontend-only auth enforcement:
  - Rejected because API endpoints still require server-side trust decisions.
- localStorage-centric token model:
  - Rejected due to SSR mismatch risk and weaker security posture for this stack.
- Separate custom JWT issuer in backend:
  - Rejected for initial scope; Supabase-issued token verification already satisfies the immediate security model and reduces complexity.

## Key Implementation Details
Planned implementation follows three phases:
1. SSR foundation in Next.js with `@supabase/ssr`, separated clients, and proxy/middleware refresh path.
2. NestJS route protection using JWKS-backed JWT verification and standardized claim extraction.
3. Hardening and operations: cookie policy alignment, auth audit logs, and troubleshooting runbook.

## Lessons Learned
A reliable auth rollout needs measurable criteria from day one. During planning, adding explicit success metrics (redirect loop target, API enforcement target, refresh reliability target) improved development readiness and QA traceability.

## Expected Results and Metrics
- Redirect loop rate on auth routes:
  - Baseline: not measured
  - Target: 0 loops in smoke tests
  - Current: pending implementation
- Protected API enforcement:
  - Baseline: not measured
  - Target: 100% expected `401`/`200` outcomes in integration tests
  - Current: pending implementation
- Session refresh reliability:
  - Baseline: not measured
  - Target: >=99% successful staged refresh checks
  - Current: pending implementation
