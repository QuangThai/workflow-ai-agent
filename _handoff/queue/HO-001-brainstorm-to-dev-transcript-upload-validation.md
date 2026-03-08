---
id: HO-001
from: brainstorm
to: dev
priority: P1
status: pending
created: 2026-03-07T00:00:00Z
spec: SPEC-001
---

# HO-001: Document & Clarify Transcript Upload Validation

## Context
Users and developers lack clear understanding of file upload validation rules for transcripts (`.txt`, `.srt`, `.vtt`). Current validation is strict but undocumented, leading to confusion about:
- What formats are supported
- Which formats require timestamps
- What happens when validation fails
- DoS protection limits

This ticket standardizes documentation and error messaging across frontend & backend.

## Scope

### Backend
- Add "§ File Upload Validation" section to `scopelytics-ai-backend/PRD.md`
- Document validation flow, formats table, HTTP status codes, and DoS limits
- Update docstrings in `src/app/services/transcript_parsers.py`

### Frontend
- Add "§ Upload Validation Rules" section to `scopelytics-ai-frontend/PRD.md`
- Add validation error message constants in `components/upload/constants.ts`
- Add detailed comments in `components/upload/hooks/useFileDropzone.ts`

### Shared (_context)
- Create decision record: `_context/decisions/upload-validation-rules.md`
- Rationale: Why `.txt` files are flexible (deferred format validation)

## Implementation Plan

### Phase 1: Backend Documentation (2-3 hours)
1. Create format validation table (ext, requirement, example, error code)
2. Document DoS limits with rationale
3. Map error codes to HTTP status codes in PRD
4. Update `transcript_parsers.py` docstrings with format rules

### Phase 2: Frontend Documentation & Code (2-3 hours)
1. Add validation rules section to frontend PRD
2. Create constants for error message templates
3. Add inline comments to `useFileDropzone.ts` explaining per-format validation
4. Verify error messages match backend status codes

### Phase 3: Decision Record & Examples (1-2 hours)
1. Create decision record in `_context/decisions/`
2. Add code examples (valid & invalid files) to PRD sections
3. Review and cross-reference both PRDs

## Deliverables
- [ ] Backend PRD: "§ File Upload Validation" added with table & status codes
- [ ] Frontend PRD: "§ Upload Validation Rules" added
- [ ] Constants file: Error message templates added
- [ ] `useFileDropzone.ts`: Format validation logic commented
- [ ] `transcript_parsers.py`: Docstring updated with format requirements
- [ ] Decision record: `_context/decisions/upload-validation-rules.md` created
- [ ] Examples: Valid & invalid sample files documented in PRDs

## Acceptance Criteria
- [ ] All supported formats documented with examples
- [ ] Error codes and HTTP status mapped in PRD
- [ ] `.txt` behavior (plain text, no timestamps) explicitly documented
- [ ] DoS limits (10k segments, 5k chars/line, 24 hours) documented
- [ ] Frontend error messages match backend error codes
- [ ] No broken links in PRDs
- [ ] Decision record explains trade-off for flexible `.txt` validation

## Dev Notes
- **Key files to modify**:
  - `scopelytics-ai-backend/PRD.md` (add section)
  - `scopelytics-ai-frontend/PRD.md` (add section)
  - `src/app/services/transcript_parsers.py` (update docstrings)
  - `components/upload/hooks/useFileDropzone.ts` (add comments)
  - `components/upload/constants.ts` (add error constants)

- **Related code** (reference, no changes):
  - `src/app/services/analysis_service.py:create_analysis_from_upload()` (line 131)
  - `src/app/api/v1/endpoints/config.py` (config exposure)

- **No migrations needed** ✓

- **No tests required** (documentation only)

## References
- Spec: `_context/specs/SPEC-001-transcript-upload-validation.md`
- Backend validation: `src/app/services/transcript_parsers.py:521-560`
- Backend parsing: `src/app/services/transcript_parsers.py:563-678`
- Frontend validation: `components/upload/hooks/useFileDropzone.ts:41-89`
- Analysis service: `src/app/services/analysis_service.py:163-291`

## Open Questions for Dev
1. Should we add MIME type validation on backend (beyond extension)?
2. Should `.txt` files have a minimum content length requirement?
3. Should we provide user-facing format detector/validator tool?

---
**Status**: Ready for dev pickup  
**Effort**: S (Small — 5-8 hours documentation)  
**Next**: Assign to `/kd-dev`
