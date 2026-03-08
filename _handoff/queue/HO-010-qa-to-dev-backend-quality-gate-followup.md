---
id: HO-010
from: qa
to: dev
priority: P0
status: pending
created: 2026-03-08
spec: N/A (ad-hoc QA for backend type-fix batch)
total_phases: 1
current_phase: 1
loop_count: 1
output_mode: full_history
---

# QA Feedback: Backend Quality Gates After Mypy Fix Batch

## Context
- No `status: done` + `to: dev` ticket was present in `_handoff/queue/` at QA start.
- QA executed a full quality-gate run on the current workspace state after the mypy-fix batch.

## QA Report
- **Date**: 2026-03-08
- **Agent**: qa
- **Verdict**: FAIL

### Automated Checks
| Check | Result |
|-------|--------|
| Backend lint (`ruff check src`) | ✅ |
| Backend format (`ruff format --check src`) | ✅ |
| Backend types (`mypy src`) | ✅ |
| Backend tests (`pytest -v`) | ❌ (23 failed, 884 passed, 1 skipped) |
| Backend security regression (`pytest tests/test_guardrails.py tests/test_guardrails_security.py tests/test_prompt_security.py -v`) | ✅ (60 passed) |
| Frontend lint (`npm run lint`) | ✅ |
| Frontend build (`npm run build`) | ✅ |
| Frontend API contract (`npm run check:api-contract`) | ✅ |

### Acceptance Criteria
| Criterion | Result | Evidence |
|-----------|--------|----------|
| Backend lint/format pass | ✅ | `ruff check src` and `ruff format --check src` succeeded. |
| Backend typecheck pass | ✅ | `mypy src` reports `Success: no issues found in 163 source files`. |
| Full backend test suite pass | ❌ | `pytest -v` ended with `23 failed, 884 passed, 1 skipped`. |
| Security regression pass | ✅ | Security subset ran green (`60 passed`). |
| Frontend quality checks pass | ✅ | lint/build/contract checks all passed. |

### Issues Found
- Full backend suite has 23 failures, including:
  - `tests/integration/test_full_pipeline.py::TestPipelineErrorScenarios::test_invalid_file_upload`
  - `tests/test_ab_testing.py::TestAnalysisCacheFlag::test_cache_default_false`
  - `tests/test_analysis.py::test_upload_binary_as_txt`
  - `tests/test_excel_export_endpoints.py::test_preview_download_with_noisy_transcript_payload`
  - `tests/test_live_quality.py::TestLiveQuality::test_product_meeting_audio_transcript_3`
  - `tests/test_metric_definitions_rubric.py` (12 failures, unicode decoding)
  - `tests/test_openai_service.py` (2 failures, disallowed model allowlist)
  - `tests/test_prompt_preamble.py::TestPreambleConfidenceCalibration::test_preamble_penalizes_missing_evidence`
  - `tests/test_transcript_parsers.py::TestTxtParser::test_parse_txt_no_timestamps`
  - `tests/test_transcript_sniffing.py::TestSniffTranscriptFormatSync::test_sniff_txt_no_timestamps`

### Notes for Dev
- Typecheck gate is fixed and green.
- Remaining blocker is backend test baseline; address failed tests before requesting release readiness.
