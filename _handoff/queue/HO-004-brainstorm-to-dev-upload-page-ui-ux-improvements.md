---
id: HO-004
from: brainstorm
to: dev
priority: P1
status: done
created: 2026-03-08T00:00:00Z
spec: SPEC-004
total_phases: 2
current_phase: 1
loop_count: 0
output_mode: full_history
---

# Upload Page UI/UX Improvements

## Contract

- **task_description**: Implement SPEC-004 Upload Page UI/UX Improvements. Fix (1) ErrorAlert dismiss label mismatch in text mode — add `dismissLabel` prop and pass "Clear text" from TextModePanel; (2) Touch target compliance — MetadataDisplay toggle and ModeToggle button must reach ≥44×44px; (3) Card bottom margin — reduce mb-16 to mb-8 or responsive; (4) Dropzone format list — ensure no horizontal scroll on 375px viewport. Phase 3 (mode discoverability) is deferred. Read the spec and research note for full context.
- **acceptance_criteria**: (1) In text mode, error dismiss button shows "Clear text". (2) MetadataDisplay "File details" toggle has ≥44×44px touch target. (3) ModeToggle button has ≥44×44px touch target. (4) Card bottom margin reduced; no excessive whitespace on 375px. (5) Dropzone format list does not cause horizontal scroll on 375px. (6) `npm run lint && npm run build` passes.
- **context_keys**: `_context/specs/SPEC-004-upload-page-ui-ux-improvements.md`, `_context/research/2026-03-08-upload-page-ui-ux-review.md`
- **output_mode**: full_history

## Context

Review of the `/upload` page against Vercel Web Interface Guidelines and ui-ux-pro-max revealed accessibility, touch-target, and UX consistency issues. This handoff implements Phase 1 and Phase 2 only. Phase 3 (mode toggle discoverability) is deferred.

## Scope

- **Backend**: None
- **Frontend**: ErrorAlert, MediaModePanel, TextModePanel, MetadataDisplay, ModeToggle, FileUploader, DropzoneEmptyState
- **Shared**: None

## Implementation Plan (Phased)

### Phase 1: Critical UX Fixes (P1)

- **Scope**: `ErrorAlert.tsx`, `MediaModePanel.tsx`, `TextModePanel.tsx`, `MetadataDisplay.tsx`, `ModeToggle.tsx`
- **Tasks**:
  1. Add optional `dismissLabel?: string` prop to ErrorAlert; default "Choose another file" when omitted
  2. MediaModePanel: pass `dismissLabel="Choose another file"` to ErrorAlert (or omit)
  3. TextModePanel: pass `dismissLabel="Clear text"` to ErrorAlert
  4. MetadataDisplay: change toggle to `min-h-[44px] min-w-[44px]` (or equivalent padding)
  5. ModeToggle: change button to `min-h-[44px]` while keeping compact style
- **Acceptance Criteria**:
  - [ ] Text mode error dismiss shows "Clear text"
  - [ ] MetadataDisplay toggle ≥44×44px
  - [ ] ModeToggle button ≥44×44px
- **Depends on**: none

### Phase 2: Layout & Polish (P2)

- **Scope**: `FileUploader.tsx`, `DropzoneEmptyState.tsx`
- **Tasks**:
  1. FileUploader: reduce Card `mb-16` to `mb-8` (or `mb-8 sm:mb-12` if preferred)
  2. DropzoneEmptyState: improve format list layout — use flex-wrap, gap, ensure no horizontal scroll on 375px
- **Acceptance Criteria**:
  - [ ] Card bottom margin reduced; no excessive whitespace on 375px
  - [ ] Dropzone format list does not cause horizontal scroll on 375px
- **Depends on**: Phase 1

## Deliverables

- [ ] ErrorAlert with dismissLabel prop
- [ ] MediaModePanel and TextModePanel passing correct dismissLabel
- [ ] MetadataDisplay touch target fix
- [ ] ModeToggle touch target fix
- [ ] FileUploader margin adjustment
- [ ] DropzoneEmptyState responsive format list
- [ ] `npm run lint && npm run build` passes

## Dev Notes

- Relevant files: `scopelytics-ai-frontend/components/upload/ErrorAlert.tsx`, `MediaModePanel.tsx`, `TextModePanel.tsx`, `MetadataDisplay.tsx`, `ModeToggle.tsx`, `FileUploader.tsx`, `DropzoneEmptyState.tsx`
- Related tests: Run frontend lint/build; manual QA on 375px viewport
- Migration needed: no

## QA Report

- **Date**: 2026-03-08
- **Agent**: qa
- **Verdict**: PASS
- **Phase**: 1 & 2 of 2

### Automated Checks

| Check | Result |
|-------|--------|
| Backend lint | N/A (no backend changes) |
| Backend types | N/A |
| Backend tests | N/A |
| Frontend lint | ✅ |
| Frontend build | ✅ |
| API contract | ⚠️ Pre-existing path config (backend vs scopelytics-ai-backend); not caused by this change |

### Acceptance Criteria

| Criterion | Result | Evidence |
|----------|--------|----------|
| Text mode error dismiss shows "Clear text" | ✅ | TextModePanel.tsx:95 `dismissLabel="Clear text"` |
| MetadataDisplay toggle ≥44×44px | ✅ | MetadataDisplay.tsx:29 `min-h-[44px] min-w-[44px]` |
| ModeToggle button ≥44×44px | ✅ | ModeToggle.tsx:36 `min-h-[44px]` |
| Card bottom margin reduced | ✅ | FileUploader.tsx:140 `mb-8 sm:mb-12` (was mb-16) |
| Dropzone format list no horizontal scroll | ✅ | DropzoneEmptyState.tsx:60 `flex flex-wrap gap-x-2 gap-y-1` |
| npm run lint && npm run build passes | ✅ | Both passed |

### Issues Found

None

### Progress Ledger

- **is_request_satisfied**: true — All acceptance criteria met; implementation matches spec
- **is_in_loop**: false — First QA cycle
- **is_progress_being_made**: N/A
- **loop_count**: 0
- **instruction_or_question**: N/A (PASS)

### Notes for Release

UI-only changes. Manual smoke test on /upload at 375px viewport recommended to confirm layout and touch targets.

---

## Implementation Log

- **Phase**: 1 & 2 of 2 (both phases implemented)
- **Files changed**: ErrorAlert.tsx, TextModePanel.tsx, MetadataDisplay.tsx, ModeToggle.tsx, FileUploader.tsx, DropzoneEmptyState.tsx
- **Tests added**: None (UI-only changes; manual QA on 375px viewport recommended)
- **Migration**: No
- **Loop iteration**: 0
- **Notes**: MediaModePanel does not pass dismissLabel (default "Choose another file" is correct for media mode). check:api-contract fails due to pre-existing path config (backend vs scopelytics-ai-backend); lint and build pass.

## References

- Spec: `_context/specs/SPEC-004-upload-page-ui-ux-improvements.md`
- Research: `_context/research/2026-03-08-upload-page-ui-ux-review.md`
- PRD: `scopelytics-ai-frontend/PRD.md` §FR-UPLOAD
