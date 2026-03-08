---
id: HO-008
from: dev
to: qa
priority: P1
status: done
created: 2026-03-08T00:00:00Z
spec: N/A (bug-fix fast path)
total_phases: 1
current_phase: 1
loop_count: 0
output_mode: full_history
---

# QA: 401 But Still Showing Account Until Reload

## Contract
- **task_description**: Verify bug fix for stale authenticated UI state when API returns 401/403. Confirm the frontend now redirects to login immediately on auth errors instead of continuing to display authenticated header/account info until manual reload.
- **acceptance_criteria**: (1) Any client-side API call returning 401/403 while user is on protected pages triggers immediate redirect to `/login?next=...`. (2) No redirect loop occurs on auth pages (`/login`, `/register`, `/forgot-password`, `/reset-password`, `/reset-password/invalid`). (3) Existing non-auth error handling remains unchanged. (4) Frontend checks pass: `npm run lint`, `npm run build`, `npm run check:api-contract`.
- **context_keys**: `scopelytics-ai-frontend/lib/api.ts`, `scopelytics-ai-frontend/proxy.ts`, `scopelytics-ai-frontend/app/api/proxy/[...path]/route.ts`
- **output_mode**: full_history

## Context

Reported symptom: user sees 401 in network/API but account info still appears until manual reload. Root cause is client UI state not reacting immediately to unauthorized responses after cookies are cleared by proxy/auth handlers.

Implemented fix adds a shared Axios response interceptor for both auth and proxy API clients so 401/403 on client routes immediately transition user to login with preserved `next` URL.

## Scope

- **Backend**: None
- **Frontend**: `lib/api.ts`
- **Shared**: None

## Deliverables

- [x] Verify interceptor catches 401/403 from `authApi` and `proxyApi`
- [x] Verify immediate redirect to login with `next` query
- [x] Verify auth pages are excluded from forced redirect
- [x] Verify no regressions in lint/build/contract checks

## Acceptance Criteria

- [x] Protected-page 401/403 causes immediate redirect to login (no reload needed)
- [x] No redirect loop while browsing auth pages
- [x] Non-401/403 errors still surface normally to existing UI handlers
- [x] `npm run lint` passes
- [x] `npm run build` passes
- [x] `npm run check:api-contract` passes

## Implementation Log

- **Files changed**: `scopelytics-ai-frontend/lib/api.ts`
- **Tests added**: None (behavioral client auth handling; verified via automated checks + QA scenario)
- **Migration**: No
- **Notes**:
  - Added global Axios response interceptor to both API clients.
  - On 401/403 (client-side only), app redirects to `/login?next=<currentPathAndQuery>`.
  - Added guard to skip redirect on auth pages and prevent duplicate concurrent redirects.

## Self-Verify (Dev)

| Check | Result |
|-------|--------|
| Frontend lint | ✅ |
| Frontend build | ✅ |
| API contract | ✅ |

## References

- `scopelytics-ai-frontend/lib/api.ts`
- `scopelytics-ai-frontend/PRD.md` §FR-AUTH-003

## QA Report
- **Date**: 2026-03-08
- **Agent**: qa
- **Verdict**: PASS-WITH-NOTES

### Automated Checks
| Check | Result |
|-------|--------|
| Backend lint | ✅ |
| Backend types | ❌ (7 pre-existing errors in unrelated backend files) |
| Backend tests | ❌ (22 failing tests, unrelated baseline failures) |
| Frontend lint | ✅ |
| Frontend build | ✅ |
| API contract | ✅ |

### Acceptance Criteria
| Criterion | Result | Evidence |
|-----------|--------|----------|
| Protected-page 401/403 causes immediate redirect to login (no reload needed) | ✅ | `scopelytics-ai-frontend/lib/api.ts` adds interceptor for both `authApi` and `proxyApi`; on status 401/403 calls `window.location.assign('/login?next=...')`. |
| No redirect loop while browsing auth pages | ✅ | Guard checks current pathname against auth pages set and skips redirect when already on auth route. |
| Non-401/403 errors still surface normally to existing UI handlers | ✅ | Interceptor only branches on 401/403 then rethrows via `Promise.reject(error)` for standard downstream handling. |
| `npm run lint` passes | ✅ | Command executed successfully in `scopelytics-ai-frontend`. |
| `npm run build` passes | ✅ | Next.js production build succeeded. |
| `npm run check:api-contract` passes | ✅ | Contract script reports `API contract check passed (9 frontend calls verified against backend routes)`. |

### Issues Found

- No scoped issues found in the auth redirect fix.
- Repository has unrelated baseline failures in backend typecheck and full backend test suite; not caused by `lib/api.ts` change.

### Notes for Release

- Safe to release this frontend auth-state fix independently.
- Recommend tracking backend baseline failures in a separate stabilization ticket.
