<!-- [agent: content] Generated from SPEC-002 -->

## [0.5.0] - 2026-03-07

### Changed
- **Faster AI analysis**: Optimized the analysis pipeline with 5 targeted latency improvements, reducing worst-case processing time from 60-180s significantly
- **Smarter token counting**: Replaced 6-7 sequential token-counting calls with a single parallel batch operation using tiktoken's `encode_ordinary_batch`
- **Tuned recall pass**: Recall pass now only triggers when feature coverage drops below 30% (was 50%) and at least 2 features are already extracted — eliminates unnecessary LLM calls on ~30% of requests
- **Reduced retry budget**: Schema and guardrail re-ask retries reduced from 2+2 to 1+1 (max 3 total LLM calls, was 5). Structured Outputs already enforces schema correctness, making extra retries rarely needed
- **Cached transcript classification**: Meeting-vs-document classification results are now cached in Redis (1h TTL), avoiding redundant LLM calls when the same transcript is retried or re-uploaded
- **Eliminated double token counting**: `_preprocess_transcript` now returns the token count it already computes internally, removing a redundant counting step on every request
