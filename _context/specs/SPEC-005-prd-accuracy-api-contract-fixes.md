---
id: SPEC-005
title: PRD Accuracy & API Contract Fixes
status: released
priority: P1
effort: S
created: 2026-03-08
author: agent:handoff-spec
---

# SPEC-005: PRD Accuracy & API Contract Fixes

## Problem

The PRD review (2026-03-08) identified discrepancies between documentation and implementation that block release gating and create contract drift:

1. **API contract check fails**: `verify-api-contract.mjs` uses `backend` path but the actual directory is `scopelytics-ai-backend`. `npm run check:api-contract` returns ENOENT and cannot run.
2. **Contract check incomplete**: The script does not verify evaluations and feedback endpoints that the frontend calls via proxy.
3. **Config API gap**: Backend `/config/upload` does not return `max_text_input_size`, `max_transcript_file_size_mb`, `allowed_transcript_formats`, `transcript_upload_enabled`. Frontend falls back to defaults even when config succeeds.
4. **PRD documentation**: Backend FR-08 does not clarify root-level vs API v1 health endpoints.

## Fact Ledger

### Verified Facts
- Backend root is `scopelytics-ai-backend`, not `backend`.
- Frontend proxy calls evaluations and feedback endpoints; script only checks auth, analysis, health, config, feature_context.
- Backend has `MAX_TEXT_CONTENT_LENGTH`, `MAX_TRANSCRIPT_FILE_SIZE_MB`, `TRANSCRIPT_UPLOAD_ENABLED`, `ALLOWED_TRANSCRIPT_FORMATS` in settings but config endpoint does not expose them.
- Root-level: `/health`, `/ready`, `/metrics`. API v1: `/api/v1/health`, `/api/v1/health/dependencies`.

### Assumptions
- No migration needed for config response extension (additive only).
- Frontend useUploadConfig will consume new fields when present; existing `??` fallbacks remain safe.

## Proposed Solution

### Phase 1: Fix API Contract Script (P0)
- Change `backendRoot` from `"backend"` to `"scopelytics-ai-backend"`.
- Add `evaluations.py` and `feedback.py` to `backendEndpointSpecs` with correct prefixes.

### Phase 2: Config API & PRD Updates (P1)
- Extend backend `/config/upload` response with four new fields.
- Update Backend PRD FR-08 to document root-level vs API v1 health endpoints.

## Technical Approach

### Backend Changes
- `src/app/api/v1/endpoints/config.py`: Add `max_text_input_size`, `max_transcript_file_size_mb`, `allowed_transcript_formats`, `transcript_upload_enabled` to `FileUploadConfigResponse` and return dict.

### Frontend Changes
- `scripts/verify-api-contract.mjs`: Fix backendRoot path; add evaluations and feedback to backendEndpointSpecs.

### Shared
- `scopelytics-ai-backend/PRD.md`: Update FR-08 with endpoint path clarification.

## Phases

### Phase 1: API Contract Script Fix
- **Scope**: `scopelytics-ai-frontend/scripts/verify-api-contract.mjs`
- **Tasks**:
  1. Change `path.resolve(frontendRoot, "..", "backend")` → `path.resolve(frontendRoot, "..", "scopelytics-ai-backend")`
  2. Add `{ file: path.join(backendRoot, "src", "app", "api", "v1", "endpoints", "evaluations.py"), prefix: "" }`
  3. Add `{ file: path.join(backendRoot, "src", "app", "api", "v1", "endpoints", "feedback.py"), prefix: "/feedback" }`
- **Acceptance Criteria**:
  - [ ] `npm run check:api-contract` passes from scopelytics-ai-frontend directory
  - [ ] Script resolves to scopelytics-ai-backend and reads evaluations.py, feedback.py
- **Depends on**: none

### Phase 2: Config API & PRD Documentation
- **Scope**: `scopelytics-ai-backend/src/app/api/v1/endpoints/config.py`, `scopelytics-ai-backend/PRD.md`
- **Tasks**:
  1. Extend `FileUploadConfigResponse` with `max_text_input_size`, `max_transcript_file_size_mb`, `allowed_transcript_formats`, `transcript_upload_enabled`
  2. Map from settings: `MAX_TEXT_CONTENT_LENGTH`, `MAX_TRANSCRIPT_FILE_SIZE_MB`, `ALLOWED_TRANSCRIPT_FORMATS`, `TRANSCRIPT_UPLOAD_ENABLED`
  3. Update PRD §7 FR-08: Add note that root-level `/health`, `/ready`, `/metrics` serve ops; API v1 `/api/v1/health`, `/api/v1/health/dependencies` serve frontend proxy
- **Acceptance Criteria**:
  - [ ] `GET /api/v1/config/upload` returns all four new fields
  - [ ] Frontend useUploadConfig receives values (no fallback when backend responds)
  - [ ] Backend PRD FR-08 documents both endpoint sets
- **Depends on**: Phase 1

## Acceptance Criteria

- [ ] `npm run check:api-contract` passes in CI and locally
- [ ] Backend config/upload returns `max_text_input_size`, `max_transcript_file_size_mb`, `allowed_transcript_formats`, `transcript_upload_enabled`
- [ ] Backend PRD FR-08 clarifies root vs API v1 health endpoints
- [ ] No regression: existing tests pass, frontend build succeeds

## Open Questions

- None (all blockers resolved in PRD review)
