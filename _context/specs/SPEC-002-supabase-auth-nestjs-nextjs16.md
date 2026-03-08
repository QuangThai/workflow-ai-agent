---
id: SPEC-002
title: Supabase Auth Integration for NestJS + Next.js 16
status: approved
priority: P1
effort: M
created: 2026-03-08
author: agent:brainstorm
---

[agent: brainstorm]

# SPEC-002: Supabase Auth Integration for NestJS + Next.js 16

## Problem
The product needs a unified authentication foundation across the SSR frontend (Next.js 16) and backend APIs (NestJS). Without a clear and consistent auth architecture, the team is likely to face client-server session mismatches, inconsistent route protection behavior, and API exposure to improperly validated tokens. In addition, incorrect cookie or refresh-token handling may cause redirect loops or unexpected logout events in production. The goal of this spec is to define a secure, standards-aligned foundation based on official Supabase, Next.js, and NestJS guidance, while preserving a clean path for future OAuth, role-based authorization, and RLS expansion without major refactoring.

## Fact Ledger
### Verified Facts
- Next.js 16 App Router recommends server-side session handling with cookie controls.
- Supabase SSR for Next.js requires separated browser/server clients and proxy or middleware-based session refresh.
- Unverified session data from cookies must not be used as the source of truth for server-side authorization.
- NestJS provides established guard and JWT verification patterns for endpoint protection.
- OWASP requires session and cookie hardening (for example: `Secure`, `HttpOnly` where applicable, `SameSite`, server-side timeout and invalidation).

### Facts to Look Up
- None unresolved. Required technical and security references were captured in `_context/research/2026-03-08-supabase-auth-nestjs-nextjs16.md`.

### Assumptions
- The project will include `apps/service-a` (NestJS API) and `apps/service-b` (Next.js 16 app). (confidence: Medium)
- A Supabase project is available with Auth enabled. (confidence: Medium)
- The backend should trust Supabase-issued tokens instead of minting a separate custom JWT system. (confidence: High)

## Service Scope
- **Service A (`apps/service-a`)**: NestJS auth verification module, guards, decorators, integration tests.
- **Service B (`apps/service-b`)**: Next.js 16 Supabase SSR clients, proxy/middleware session refresh, auth routes/actions.
- **Shared Context**: `_context/specs`, `_context/research`, `_context/product-state`.

## Success Metrics
- **Metric 1: Redirect loop rate on auth routes**
  - Baseline: Not measured; auth flow not implemented.
  - Target: `0` redirect loops in smoke tests across login, callback, and protected-route navigation.
- **Metric 2: Protected API auth enforcement**
  - Baseline: Not measured; Supabase JWT verification not implemented in NestJS.
  - Target: `100%` of protected endpoints return `401` for missing/invalid token and `200` for valid token in integration tests.
- **Metric 3: Session refresh reliability**
  - Baseline: Not measured; SSR session refresh middleware/proxy not implemented.
  - Target: `>= 99%` successful session refresh behavior in staged verification runs (no unexpected logout during normal navigation).

## Risk and Rollback
- **Risk Level**: medium
- **Global Rollback Strategy**: Disable strict auth enforcement via feature/config flags, revert auth integration files by phase, and temporarily restore public/internal access paths until verification is restored.

## Proposed Solution
Adopt a hybrid authentication boundary:
- Next.js 16 manages sign-in/sign-up/callback/logout flows and SSR session lifecycle using `@supabase/ssr` with proxy or middleware token refresh.
- NestJS accepts only valid Supabase bearer JWTs, verifies signature and claims, then enforces route and role policies at the API layer.
- Authorization follows a defense-in-depth model: frontend route checks are UX-oriented, while data security decisions remain in NestJS and/or RLS.

## Technical Approach
### Backend Changes
- Create a NestJS auth module to verify Supabase JWTs through JWKS.
- Implement `AuthGuard` and optional `RolesGuard`, plus decorators for public/protected routes.
- Normalize request user context from JWT claims (`sub`, `role`, and optional email).
- Add tests for unauthorized, expired-token, and valid-token scenarios.

### Frontend Changes
- Implement Supabase browser/server client utilities using `@supabase/ssr`.
- Add `proxy.ts` (or framework-aligned middleware) to refresh sessions and synchronize request/response cookies.
- Implement auth routes/actions for login, sign-up, callback code exchange, and logout.
- Protect private pages with server-side checks, prioritizing authoritative verification before rendering sensitive data.

## Phase Planning Rules
- Use numbered phases: `Phase 1`, `Phase 2`, ...
- Inside each phase, tasks MUST use hierarchical numbering:
  - Phase 1 tasks: `1.1`, `1.2`, `1.3`, ...
  - Phase 2 tasks: `2.1`, `2.2`, `2.3`, ...
- Keep each task atomic and testable in one dev iteration.
- Every task must map to at least one acceptance criterion.
- If effort is `M` or `L`, phase breakdown is mandatory.

## Phases (required for effort ≥ M)
### Phase 1: Next.js SSR Auth Foundation
- Objective: Establish a reliable Supabase SSR session lifecycle in the frontend.
- Scope: `apps/service-b` auth utilities, proxy/middleware, auth routes/actions.
- Tasks:
  - [ ] **1.1** Implement `lib/supabase/client` and `lib/supabase/server` using `@supabase/ssr`.
  - [ ] **1.2** Implement `proxy.ts`/middleware to refresh sessions and synchronize request/response cookies.
  - [ ] **1.3** Implement login/callback/logout flow with post-login redirects that avoid prefetch race conditions.
- Acceptance Criteria:
  - [ ] Login-to-protected-page flow is stable in local and production-like environments without redirect loops.
  - [ ] Protected pages do not expose sensitive data when the session is missing or invalid.
- Dependencies: none
- Risks: Cookie attribute mismatches between local development and production HTTPS.
- Rollback: Disable proxy-based auth protection, temporarily keep routes public, and revert auth utility files.

### Phase 2: NestJS JWT Verification + Route Protection
- Objective: Protect APIs with reliable Supabase token validation.
- Scope: `apps/service-a` auth module, guards, decorators, tests.
- Tasks:
  - [ ] **2.1** Implement a JWKS-based JWT verification service and standardized claim parsing.
  - [ ] **2.2** Implement `AuthGuard` plus route metadata (`@Public`, optional role metadata).
  - [ ] **2.3** Add integration tests for unauthorized, expired-token, and valid-token paths.
- Acceptance Criteria:
  - [ ] Protected API routes return `401` for missing/invalid tokens and `200` for valid tokens.
  - [ ] User claims are correctly attached to the request context for downstream handlers.
- Dependencies: Phase 1 (token issuance/session behavior validated)
- Risks: Token verification failures during signing-key rotation if JWKS cache settings are misconfigured.
- Rollback: Put guard enforcement behind feature flags in monitor mode and temporarily open selected internal routes.

### Phase 3: Security Hardening + Operational Readiness
- Objective: Finalize security baseline and production operability for auth.
- Scope: frontend/backend configuration, logging, operational runbook.
- Tasks:
  - [ ] **3.1** Standardize cookie, security-header, and session-timeout policy per environment.
  - [ ] **3.2** Add audit logs for login/logout/auth failures and an auth-error monitoring dashboard.
  - [ ] **3.3** Document a troubleshooting runbook (refresh failures, clock skew, JWKS cache, callback issues).
- Acceptance Criteria:
  - [ ] Auth security checklist meets internal baseline and core OWASP session controls.
  - [ ] Operational documentation enables fast diagnosis of the top 5 production auth failure patterns.
- Dependencies: Phase 1, Phase 2
- Risks: Overly strict hardening may degrade UX (for example, premature session drops).
- Rollback: Relax hardening through configuration toggles while preserving the core token-verification path.

## Acceptance Criteria
- [ ] Next.js 16 Supabase SSR auth flow is stable for login, refresh, and logout.
- [ ] NestJS verifies Supabase bearer tokens and protects endpoints according to policy.
- [ ] Cookie/session/JWT security baseline is implemented and validated through tests/checklists.

## Open Questions
- [ ] nice-to-have: Should OAuth providers (Google/GitHub) be included immediately after the foundation phases?
- [ ] nice-to-have: Should Postgres RLS be enabled in the same release or moved to a follow-up SPEC?
- [ ] nice-to-have: Is a multi-tenant JWT claim model required in the first release?
