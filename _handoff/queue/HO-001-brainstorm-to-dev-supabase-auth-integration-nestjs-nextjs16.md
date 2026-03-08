---
id: HO-001
from: brainstorm
to: dev
priority: P1
status: blocked
created: 2026-03-08T12:41:45Z
spec: SPEC-002
total_phases: 3
current_phase: 1
loop_count: 0
output_mode: full_history
service_scope: apps/service-a, apps/service-b
risk_level: medium
rollback_plan: Disable strict auth enforcement via config flags and revert phase-specific auth files.
---

# Supabase Auth Integration for NestJS + Next.js 16

## Contract
- **task_description**: Implement Phase 1 of SPEC-002 by establishing the Supabase SSR auth foundation in Next.js 16 (`apps/service-b`). Create browser/server Supabase clients, proxy/middleware-based session refresh, and login/callback/logout flow. Keep implementation aligned with official Supabase SSR guidance and ensure no redirect loops or sensitive data leakage on protected pages.
- **acceptance_criteria**: (1) Login-to-protected flow is stable without redirect loops in local and production-like runs. (2) Protected pages do not render sensitive data when session is missing/invalid. (3) Phase 1 tasks `1.1`-`1.3` are complete and test evidence is captured.
- **context_keys**: `_context/specs/SPEC-002-supabase-auth-nestjs-nextjs16.md`, `_context/research/2026-03-08-supabase-auth-nestjs-nextjs16.md`, `_context/product-state.md`, `_context/lessons.md`
- **output_mode**: full_history

## Context
This feature introduces a unified authentication foundation across Next.js 16 (SSR frontend) and NestJS (backend APIs) using Supabase as the identity provider. The approved design follows a hybrid boundary: frontend manages session lifecycle and auth UX, backend verifies bearer tokens and enforces API authorization.

## Scope
- **Backend**: `apps/service-a` JWT verification and guards are defined in later phases (Phase 2+).
- **Frontend**: `apps/service-b` Supabase SSR client utilities, proxy/middleware session refresh, auth actions/routes.
- **Shared**: Update `_context` artifacts as work progresses and preserve phase-level traceability.

## Implementation Plan (Phased)

### Phase 1: Next.js SSR Auth Foundation — high
- **Scope**: `apps/service-b/lib/supabase/client.ts`, `apps/service-b/lib/supabase/server.ts`, `apps/service-b/proxy.ts` (or `middleware.ts` depending on app structure), auth actions/routes under `apps/service-b/app/**`.
- **Objective**: Establish reliable Supabase SSR session lifecycle in the frontend.
- **Tasks**:
  - [ ] **1.1** Implement `lib/supabase/client` and `lib/supabase/server` using `@supabase/ssr`.
  - [ ] **1.2** Implement proxy/middleware session refresh with synchronized request/response cookies.
  - [ ] **1.3** Implement login/callback/logout flow with post-login redirect behavior that avoids prefetch race issues.
- **Acceptance Criteria**:
  - [ ] Stable login-to-protected navigation with no redirect loops (maps: `1.2`, `1.3`).
  - [ ] Protected pages block sensitive data when session is missing or invalid (maps: `1.2`, `1.3`).
  - [ ] Supabase SSR utilities are reusable and imported through a single consistent pattern (maps: `1.1`).
- **Depends on**: none
- **Rollback**: Disable proxy/middleware auth enforcement and revert newly added Supabase client/auth route files.

### Phase 2: NestJS JWT Verification + Route Protection — high
- **Scope**: `apps/service-a/src/auth/**`, guard/decorator wiring in protected modules, auth tests in `apps/service-a/test/**`.
- **Objective**: Protect APIs with reliable Supabase token validation.
- **Tasks**:
  - [ ] **2.1** Implement JWKS-based JWT verification service with standardized claim parsing.
  - [ ] **2.2** Implement `AuthGuard` and route metadata (`@Public`, optional role metadata).
  - [ ] **2.3** Add integration tests for unauthorized, expired-token, and valid-token paths.
- **Acceptance Criteria**:
  - [ ] Protected API routes return `401` for missing/invalid token and `200` for valid token (maps: `2.1`, `2.2`, `2.3`).
  - [ ] User claims are attached to request context for downstream handlers (maps: `2.1`, `2.2`).
- **Depends on**: Phase 1
- **Rollback**: Feature-flag guard enforcement to monitor mode and temporarily open selected internal routes.

### Phase 3: Security Hardening + Operational Readiness — medium
- **Scope**: security config/docs/logging in both services; operational docs in `_context` and service docs.
- **Objective**: Finalize security baseline and production operability.
- **Tasks**:
  - [ ] **3.1** Standardize cookie/security-header/session-timeout policy per environment.
  - [ ] **3.2** Add audit logging for login/logout/auth failures and auth-error monitoring hooks.
  - [ ] **3.3** Document a troubleshooting runbook for refresh failures, clock skew, JWKS cache, and callback issues.
- **Acceptance Criteria**:
  - [ ] Auth security checklist satisfies core OWASP session controls (maps: `3.1`, `3.2`).
  - [ ] Runbook supports rapid diagnosis of top production auth failure patterns (maps: `3.3`).
- **Depends on**: Phase 1, Phase 2
- **Rollback**: Relax hardening via config toggles while keeping core token verification active.

## Deliverables
- [ ] Backend implementation (per phase)
- [ ] Frontend implementation (per phase)
- [ ] Tests (unit + integration, per phase)
- [ ] PRD.md updates if new endpoints/features are introduced

## Dev Notes
- Relevant files: `apps/service-b/lib/supabase/*`, `apps/service-b/proxy.ts` or `apps/service-b/middleware.ts`, `apps/service-b/app/**`, `apps/service-a/src/auth/**`, `apps/service-a/test/**`
- Related tests: frontend auth flow tests and backend auth integration tests under service-level test directories
- Migration needed: no database migration required for Phase 1 baseline

## References
- Spec: `_context/specs/SPEC-002-supabase-auth-nestjs-nextjs16.md`
- Research: `_context/research/2026-03-08-supabase-auth-nestjs-nextjs16.md`
- PRD: `apps/service-a/PRD.md` §TBD (file not found in current workspace)
- PRD: `apps/service-b/PRD.md` §TBD (file not found in current workspace)

## Implementation Log
- Phase: 1 of 3
- Status: blocked
- Files changed: none
- Tests added: none
- Migration: no
- Loop iteration: 0
- QA feedback addressed: n/a
- Notes: Required service repositories are missing in this workspace (`apps/`, including `apps/service-b`). Implementation cannot proceed until the target codebase is available.
