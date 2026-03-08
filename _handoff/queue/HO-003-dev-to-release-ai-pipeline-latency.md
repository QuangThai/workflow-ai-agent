---
id: HO-003
from: dev
to: release
priority: P0
status: pending
created: 2026-03-07
spec: SPEC-002
---

# Release: AI Analysis Pipeline Latency Optimization

## Summary
Five targeted backend optimizations to reduce AI analysis pipeline latency (worst-case 60-180s → significantly lower). Eliminates duplicate token counting via batch API, raises recall pass threshold to reduce unnecessary LLM calls, halves re-ask retry budget, caches classification results in Redis, and removes double token counting in the preprocess flow.

## Changes
### Backend
- `src/app/services/openai_service.py` — Added `_count_tokens_batch()` using tiktoken `encode_ordinary_batch` for parallel counting; deduplicated 6-7 sequential `_count_tokens` calls to 1 batch call in `analyze_transcript`; `_preprocess_transcript` now returns token count as third tuple element
- `src/app/services/analysis_ai_service.py` — `_needs_recall_pass()` threshold changed from `coverage < 0.5` → `coverage < 0.3` with `len(features) >= 2` guard
- `src/app/core/config.py` — `STRUCTURED_REASK_MAX_RETRIES` default `2` → `1`; `GUARDRAIL_REASK_MAX_RETRIES` default `2` → `1`
- `src/app/services/analysis_pipeline.py` — Added `_cached_classify_meeting_transcript()` with Redis cache (SHA256 key, 1h TTL)

### Frontend
- None — backend-only optimization

### Database
- Migration: No
- Migration name: N/A

## QA Status
- All automated checks: ✅
- Acceptance criteria: ✅ (AC-1 through AC-6 verified; AC-7 latency benchmark deferred to post-deploy)
- QA report: `_handoff/archive/HO-002-brainstorm-to-dev-ai-pipeline-latency.md`
- QA verdict: **PASS-WITH-NOTES**

## Deploy Notes
- **Environment variables**: No new env vars. Existing `STRUCTURED_REASK_MAX_RETRIES` and `GUARDRAIL_REASK_MAX_RETRIES` can override new defaults (1) if needed.
- **Migration steps**: None
- **Breaking changes**: None — all changes are internal behavior optimizations
- **Rollback plan**: Revert the 4 changed files. Re-ask retries can be restored to 2 via env vars without code change: `STRUCTURED_REASK_MAX_RETRIES=2 GUARDRAIL_REASK_MAX_RETRIES=2`

## Post-Deploy Verification
- [ ] Health check passes (`/health` endpoint)
- [ ] Smoke test: Upload a short transcript and verify analysis completes successfully
- [ ] Smoke test: Upload same transcript again — verify classification cache HIT in logs (`Classification cache HIT`)
- [ ] Monitor `schema_fail` and `reask_exhausted` metrics for 24h (Fix 3 reduced retries)
- [ ] Benchmark: Run 3 sample transcripts (short <1k words, medium 3-5k, long >6k) — compare `ai_stage_timings` to baseline, target ≥15% p50 reduction (AC-7)
- [ ] Monitor recall pass trigger rate — should decrease with new 30% threshold (was 50%)
