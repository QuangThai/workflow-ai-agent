---
id: HO-009
from: dev
to: release
priority: P1
status: pending
created: 2026-03-08T00:00:00Z
spec: N/A (bug-fix fast path)
total_phases: 1
current_phase: 1
loop_count: 0
output_mode: last_message
---

# Release: Auth 401 Immediate Redirect Fix

## Summary
This release fixes an auth state inconsistency where users could briefly see authenticated account UI after API 401/403 responses until a manual reload. A shared Axios interceptor now redirects immediately to `/login?next=...` on unauthorized responses for client-side auth/proxy calls, preventing stale authenticated UI.

## Changes
### Backend
- None.

### Frontend
- `scopelytics-ai-frontend/lib/api.ts` — Added global response interceptors for `authApi` and `proxyApi`.
- `scopelytics-ai-frontend/lib/api.ts` — Added guarded redirect logic for 401/403 with `next` preservation and auth-page loop prevention.

### Database
- Migration: No
- Migration name: N/A

## QA Status
- Automated checks (scoped frontend + security regression): ✅
- Acceptance criteria: ✅
- QA report: `_handoff/archive/HO-008-dev-to-qa-auth-401-stale-session-ui.md`
- Notes: Backend baseline still has unrelated existing failures (types/tests), unchanged by this release.

## Deploy Notes
- Environment variables: None added
- Migration steps: None
- Breaking changes: None
- Rollback plan: Revert `scopelytics-ai-frontend/lib/api.ts` to previous commit/version to restore prior client behavior.

## Post-Deploy Verification
- [ ] Health check passes (`/api/proxy/health`)
- [ ] Smoke test: expire/revoke session on `/dashboard`, verify immediate redirect to `/login?next=...` without reload
- [ ] Monitor: spike in frontend auth redirects or unexpected login loop reports
