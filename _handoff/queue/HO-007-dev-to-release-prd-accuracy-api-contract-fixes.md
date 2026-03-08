---
id: HO-007
from: dev
to: release
priority: P0
status: pending
created: 2026-03-08
spec: SPEC-005
total_phases: 2
current_phase: 2
loop_count: 0
output_mode: last_message
---

# Release: PRD Accuracy & API Contract Fixes

## Contract
- **task_description**: Deploy PRD Accuracy & API Contract Fixes to production. Run pre-deploy checks, present deploy command, verify post-deploy health.
- **acceptance_criteria**: All health checks pass, no errors in docker logs, smoke tests green. `npm run check:api-contract` passes in CI.
- **context_keys**: `_context/specs/SPEC-005-prd-accuracy-api-contract-fixes.md`, `_handoff/archive/HO-006-brainstorm-to-dev-prd-accuracy-api-contract-fixes.md`
- **output_mode**: last_message

## Summary
Fixes PRD–implementation alignment: (1) API contract verification script now uses correct backend path (`scopelytics-ai-backend`) and verifies evaluations + feedback endpoints; (2) Backend `/config/upload` returns `max_text_input_size`, `max_transcript_file_size_mb`, `allowed_transcript_formats`, `transcript_upload_enabled` so frontend receives backend validation rules; (3) Backend PRD FR-08 documents root-level vs API v1 health endpoints.

## Changes

### Backend
- `src/app/api/v1/endpoints/config.py` — Extended `FileUploadConfigResponse` with 4 new fields mapped from settings
- `PRD.md` — FR-08 clarified: root-level `/health`, `/ready`, `/metrics` for ops; frontend proxy uses `/api/v1/health`, `/api/v1/health/dependencies`

### Frontend
- `scripts/verify-api-contract.mjs` — `backendRoot` path fix (`backend` → `scopelytics-ai-backend`), added evaluations.py and feedback.py to `backendEndpointSpecs`

### Database
- Migration: No
- Migration name: N/A

## QA Status
- All automated checks: ✅ (mypy has 7 pre-existing errors in unrelated files — not in scope)
- Acceptance criteria: ✅
- QA report: `_handoff/archive/HO-006-brainstorm-to-dev-prd-accuracy-api-contract-fixes.md` §QA Report

## Deploy Notes
- Environment variables: None new
- Migration steps: None
- Breaking changes: None. Config API response is additive (new fields only). Frontend already expects these fields with fallback.
- Rollback plan: Revert commits for config.py, verify-api-contract.mjs, PRD.md. No DB rollback needed.

## Post-Deploy Verification
- [ ] Health check: `GET /health` and `GET /api/v1/health/dependencies` return 200
- [ ] Smoke test: `cd scopelytics-ai-frontend && npm run check:api-contract` exits 0
- [ ] Monitor: No new metrics; verify config/upload returns 9 fields in production
