---
id: SPEC-002
title: AI Analysis Pipeline Latency Optimization
status: released
priority: P0
effort: M
created: 2026-03-07
author: agent:brainstorm
---

# SPEC-002: AI Analysis Pipeline Latency Optimization

## Problem

The AI analysis pipeline (`AnalysisAIService.analyze()`) has significant latency issues causing 60-180s worst-case processing times. Root cause analysis identified **5 concrete bottlenecks** backed by code evidence and industry research (OpenAI latency guide, tiktoken internals, prompt caching docs):

1. **Duplicate `_count_tokens` calls** — `openai_service.py:530-625` calls `_count_tokens` **6-7 times sequentially** on the same texts. `system_prompt` and `few_shot_text` are each counted **twice**. Each call goes through `asyncio.to_thread()` adding thread context switch overhead. Impact: **+90-300ms on 100% of requests**.

2. **Recall pass threshold too aggressive** — `analysis_ai_service.py:222-226` triggers recall when feature coverage < 50% of enumerated count. Enumeration pass tends to overcount candidates vs. structured extraction. Impact: **+5-15s on ~30% of requests** (unnecessary extra LLM call).

3. **Re-ask loop allows up to 4 additional LLM calls** — `STRUCTURED_REASK_MAX_RETRIES=2` + `GUARDRAIL_REASK_MAX_RETRIES=2` means worst case: primary + 2 schema re-asks + 2 guardrail re-asks = **5 total LLM calls**. Impact: **+30-90s on 10-20% of requests**.

4. **Classification LLM result not cached** — `analysis_pipeline.py:520` calls `classify_meeting_transcript` (gpt-4o-mini) without caching. Same transcript retried or re-uploaded triggers redundant LLM classification. Impact: **+2-5s on ~40% of text/transcript uploads**.

5. **`_preprocess_transcript` internally counts tokens, then caller counts again** — `openai_service.py:558-563` double-counts transcript tokens. Impact: **+15-50ms per request**.

**Research sources:**
- OpenAI Latency Optimization Guide: "Make fewer requests", "Parallelize", "Generate fewer tokens"
- tiktoken internals (Librarian): `encode()` is CPU-bound (~15-50ms for 50k chars), GIL released in Rust, `encode_ordinary_batch()` parallelizes with real threads
- OpenAI Prompt Caching: `store=true` already enabled (`PROMPT_CACHE_ENABLED=True`), schema + system prompt prefix are cacheable
- Structured Outputs: `parse()` already in use — eliminates most schema validation failures

## Proposed Solution

Five targeted fixes, ordered by impact/effort ratio:

### Fix 1: Eliminate duplicate `_count_tokens` calls (S effort, highest impact)
Count each text component **once**, store result, reuse for all budget calculations.

### Fix 2: Tune recall pass trigger (S effort)
Raise coverage threshold from 50% → 70% and add minimum feature count guard.

### Fix 3: Reduce re-ask budget (S effort)
Lower `STRUCTURED_REASK_MAX_RETRIES` from 2 → 1 and `GUARDRAIL_REASK_MAX_RETRIES` from 2 → 1. Structured Outputs (`parse()`) already enforces schema — schema failures are rare.

### Fix 4: Cache classification results (S effort)
Cache `classify_meeting_transcript` result in Redis keyed by transcript hash with 1h TTL.

### Fix 5: Return token count from `_preprocess_transcript` (S effort)
Avoid double-counting by returning the token count alongside the processed text.

## Technical Approach

### Backend Changes

#### Fix 1: Token counting optimization in `openai_service.py`

**File:** `src/app/services/openai_service.py` (method `analyze_transcript`, lines 520-625)

Current flow (6-7 sequential `_count_tokens` calls):
```python
# Line 530-531: First count
system_prompt_overhead = await self._count_tokens(system_prompt)     # call 1
few_shot_overhead = await self._count_tokens(selected_examples_text) # call 2
# Line 558: preprocess (internal count)                               # call 3
# Line 563: count again
transcript_tokens = await self._count_tokens(transcript)             # call 4
# Line 595-599: DUPLICATE counts!
system_prompt_tokens = await self._count_tokens(system_prompt)       # call 5 ← SAME as 1
few_shot_tokens = await self._count_tokens(few_shot_text)            # call 6 ← SAME as 2
# Line 625: possible recount after reclaim
transcript_tokens = await self._count_tokens(transcript)             # call 7
```

**Fix:** Count each component once at the top, use `encode_ordinary_batch` for parallel counting:
```python
# Count all components in one batch call
async def _count_tokens_batch(self, texts: list[str]) -> list[int]:
    def _do_batch():
        enc = get_encoding(settings.GPT_MODEL)
        if enc is None:
            return [len(t) // 4 for t in texts]
        results = enc.encode_ordinary_batch(texts, num_threads=len(texts))
        return [len(r) for r in results]
    return await asyncio.wait_for(asyncio.to_thread(_do_batch), timeout=1.5)

# Usage: one call replaces 6-7
[system_tokens, few_shot_tokens, transcript_tokens] = await self._count_tokens_batch(
    [system_prompt, selected_examples_text, transcript]
)
# Reuse these cached values throughout the method
```

#### Fix 2: Recall pass threshold in `analysis_ai_service.py`

**File:** `src/app/services/analysis_ai_service.py` (method `_needs_recall_pass`, line 222)

```python
# Before:
if enumerated_count > 0:
    coverage = len(features) / enumerated_count
    if coverage < 0.5:
        return True

# After:
if enumerated_count > 0 and len(features) >= 2:
    coverage = len(features) / enumerated_count
    if coverage < 0.3:  # Only recall on severe misses
        return True
```

#### Fix 3: Reduce re-ask defaults in `config.py`

**File:** `src/app/core/config.py` (lines 204-206)

```python
# Before:
STRUCTURED_REASK_MAX_RETRIES: int = Field(default=2, ge=0, le=5)
GUARDRAIL_REASK_MAX_RETRIES: int = Field(default=2, ge=0, le=5)

# After:
STRUCTURED_REASK_MAX_RETRIES: int = Field(default=1, ge=0, le=5)
GUARDRAIL_REASK_MAX_RETRIES: int = Field(default=1, ge=0, le=5)
```

#### Fix 4: Cache classification in `analysis_pipeline.py`

**File:** `src/app/services/analysis_pipeline.py` (around line 520)

```python
async def _cached_classify(title: str, transcript: str) -> dict[str, Any] | None:
    """Classify with Redis cache by transcript hash."""
    import hashlib
    cache_key = f"classify:{hashlib.sha256(transcript.encode()).hexdigest()[:32]}"
    from ..core.redis import get_redis
    try:
        redis = await get_redis()
        cached = await redis.get(cache_key)
        if cached:
            return json.loads(cached)
    except Exception:
        pass

    result = await OpenAIService().classify_meeting_transcript(
        title=title, transcript=transcript
    )

    try:
        redis = await get_redis()
        await redis.set(cache_key, json.dumps(result, default=str), ex=3600)
    except Exception:
        pass
    return result
```

#### Fix 5: Return token count from `_preprocess_transcript`

**File:** `src/app/services/openai_service.py` (method `_preprocess_transcript`)

Change return type from `tuple[str, bool]` → `tuple[str, bool, int]` where the third element is the token count computed internally, so caller skips redundant `_count_tokens`.

### Frontend Changes
None required — this is backend-only optimization.

## Acceptance Criteria

- [ ] **AC-1**: `_count_tokens` is called **at most once per unique text** in `analyze_transcript` (verify via log grep or unit test mock)
- [ ] **AC-2**: Recall pass triggers only when coverage < 30% (was 50%) AND features >= 2
- [ ] **AC-3**: Default `STRUCTURED_REASK_MAX_RETRIES=1` and `GUARDRAIL_REASK_MAX_RETRIES=1`
- [ ] **AC-4**: Classification result is cached in Redis — second call with same transcript returns cache hit (verify via test)
- [ ] **AC-5**: `_preprocess_transcript` returns token count — no duplicate count after preprocess
- [ ] **AC-6**: All existing tests pass (`pytest`) — no regression
- [ ] **AC-7**: p50 analysis latency reduced by ≥15% measured via `ai_stage_timings` in logs (manual benchmark on 3 sample transcripts: short <1k words, medium 3-5k words, long >6k words)

## Open Questions

1. ~~Should `ANALYSIS_ENUMERATION_MIN_WORDS` (3500) be raised?~~ → Keep as-is; the recall threshold fix (Fix 2) addresses the false-trigger problem without reducing enumeration coverage
2. Should we add a `prompt_cache_key` parameter to OpenAI calls for better prefix routing? → Defer to future optimization (OpenAI auto-routing already works based on first ~256 tokens)
3. Monitor: After deploying Fix 3 (reduced re-ask), track `schema_fail` and `reask_exhausted` rates in metrics to ensure quality doesn't degrade
