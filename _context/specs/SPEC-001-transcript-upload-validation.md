---
id: SPEC-001
title: Clarify & Document Transcript Upload Validation Rules
status: approved
priority: P1
effort: S
created: 2026-03-07
author: agent:brainstorm
---

# SPEC-001: Clarify & Document Transcript Upload Validation Rules

## Problem
Current file upload validation for transcripts (`.txt`, `.srt`, `.vtt`, `.vrt`) exists but is unclear to developers and users:
- Users don't understand what validation happens at frontend vs backend
- `.txt` format accepts plain text without timestamps, but this isn't documented
- Error messages differ between frontend and backend
- Content sniffing (magic-byte validation) is not documented in PRD
- DoS protection limits (segments, line length, time span) are buried in code

**Impact**: 
- User confusion when `.txt` file is rejected
- Inconsistent error messaging UX
- No API contract documentation for rejected files

## Proposed Solution
1. **Document validation rules** in frontend/backend PRDs with examples
2. **Standardize error messages** between frontend & backend
3. **Add validation summary** to API contract
4. **Create user-facing docs** for supported formats

## Technical Approach

### Backend Changes
**File**: `scopelytics-ai-backend/PRD.md`
- Add new section "§ File Upload Validation" (after upload section)
- Document validation flow: extension → size → content sniffing → parsing
- List all supported formats with content requirements
- Document DoS limits: max 10k segments, 5k chars/line, 24 hours span
- Map error codes to HTTP status codes:
  - `unsupported_file_type` → 415 (Unsupported Media Type)
  - `file_too_large` → 413 (Payload Too Large)
  - `transcript_format_invalid` → 422 (Unprocessable Content)
  - `transcript_parse_failed` → 422 (Unprocessable Content)

**File**: `scopelytics-ai-backend/src/app/services/transcript_parsers.py`
- Add docstring header with format rules
- Example: `.txt` = plain text (no timestamps required), `.srt` = SubRip with timestamps, `.vtt` = WebVTT with WEBVTT header

### Frontend Changes
**File**: `scopelytics-ai-frontend/PRD.md`
- Add section "§ Upload Validation Rules"
- Document frontend-side validation before upload
- Show example error messages

**File**: `scopelytics-ai-frontend/components/upload/constants.ts`
- Add constants for validation error message templates
- Example: `TRANSCRIPT_FORMAT_REQUIRED_TIMESTAMP` for .srt/.vtt
- Example: `TRANSCRIPT_FORMAT_PLAIN_TEXT_OK` for .txt

**File**: `scopelytics-ai-frontend/components/upload/hooks/useFileDropzone.ts`
- Add detailed comments explaining validation logic per file type
- Link to PRD section for user reference

### Shared Changes
**File**: `_context/decisions/upload-validation-rules.md` (NEW)
- Decision: Deferred format validation for `.txt` (allow any plain text, let AI classify)
- Rationale: Flexibility for user-provided meeting notes or transcripts
- Trade-off: Some invalid files accepted, rejected only if AI fails to parse

## Acceptance Criteria

### Documentation
- [ ] Backend PRD section "§ File Upload Validation" added with:
  - Validation flow diagram or list
  - Table of formats (ext, requirement, example, error code)
  - DoS limits documented with rationale
- [ ] Frontend PRD section "§ Upload Validation Rules" added
- [ ] Decision record `upload-validation-rules.md` created in `_context/decisions/`

### Code Comments
- [ ] `transcript_parsers.py` header docstring updated with format rules
- [ ] `useFileDropzone.ts` validation logic commented per file type
- [ ] Constants file has descriptive error message keys

### Error Message Alignment
- [ ] Frontend error messages match backend status codes
- [ ] All error paths have user-friendly descriptions
- [ ] No "unknown error" messages; all failures are explained

### Examples Added
- [ ] Valid `.txt` example (plain text, no timestamps)
- [ ] Valid `.srt` example (with timestamps)
- [ ] Valid `.vtt` example (with WEBVTT header)
- [ ] Invalid `.txt` example (binary file)
- [ ] Invalid `.srt` example (missing timestamps)

## Open Questions
1. Should `.txt` files have a minimum content length requirement? Currently, any non-empty text is accepted.
2. Should we add MIME type validation on backend beyond extension check?
3. Should we provide a "format detector" tool in frontend to help users fix rejected files?

## References
- Current validation: `src/app/services/transcript_parsers.py:sniff_transcript_format()` (line 521)
- Current parsing: `src/app/services/transcript_parsers.py:parse_transcript_file()` (line 563)
- Frontend validation hook: `components/upload/hooks/useFileDropzone.ts`
- Config exposure: `src/app/api/v1/endpoints/config.py`
