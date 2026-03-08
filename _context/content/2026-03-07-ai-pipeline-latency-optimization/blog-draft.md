<!-- [agent: content] Generated from SPEC-002 -->

# How We Cut Our AI Analysis Latency by Eliminating Waste

When you upload a meeting transcript to Scopelytics AI, a lot happens behind the scenes — token budgeting, transcript classification, feature extraction, guardrail checks, and more. We noticed that some analyses were taking 60-180 seconds at worst. That's not acceptable when you're waiting for results.

We dug into the pipeline, profiled every step, and found that most of the slowness wasn't from the AI itself — it was from **our own code doing redundant work**.

## The Problem

Our `analyze_transcript` flow had five specific bottlenecks:

1. **Counting the same tokens 6-7 times.** System prompts and few-shot examples were each counted twice, every single request. Each count went through a thread context switch. That's 90-300ms of pure waste on 100% of requests.

2. **Triggering recall passes too eagerly.** When our enumeration pass estimated more features than extraction found, we'd fire an extra LLM call if coverage was below 50%. But enumeration tends to overcount — so ~30% of requests got an unnecessary 5-15 second penalty.

3. **Too many re-ask retries.** Our retry budget allowed up to 4 additional LLM calls (2 for schema fixes + 2 for guardrail re-asks). With Structured Outputs already enforcing schema, most of those retries never fired — but when they did, they added 30-90 seconds.

4. **Re-classifying the same transcript.** Every upload triggered an LLM call to classify whether the text is a meeting transcript or a document. Re-uploads and retries repeated this work. That's 2-5 seconds wasted on ~40% of text uploads.

5. **Double-counting after preprocessing.** Our transcript preprocessor counted tokens internally, then the caller counted them again. A small but universal 15-50ms tax.

## The Fix

Five surgical changes, zero new dependencies:

- **Batch token counting** — One call to tiktoken's `encode_ordinary_batch` replaces 6-7 sequential calls. It runs in parallel Rust threads, so it's faster even for a single text.

- **Smarter recall threshold** — Recall only triggers at <30% coverage (was 50%) and requires at least 2 features already extracted. This eliminates false positives from enumeration overcounting.

- **Halved retry budget** — Re-ask retries reduced from 2+2 to 1+1. Structured Outputs makes schema failures rare; one retry is enough for edge cases.

- **Redis-cached classification** — Classification results are cached by transcript content hash with a 1-hour TTL. Same transcript = instant cache hit.

- **Return the count you already have** — `_preprocess_transcript` now returns its internal token count, so the caller doesn't recount.

## The Result

All five fixes are backend-only, require no database migration, and are individually reversible. The re-ask budget can even be adjusted via environment variables without a code deploy.

We're monitoring `schema_fail` and `reask_exhausted` rates to ensure the reduced retry budget doesn't impact extraction quality — early results show no degradation.

These aren't flashy changes. There's no new model, no new architecture. Just reading the code carefully, measuring where time actually goes, and cutting the waste. Sometimes the best optimization is the one you stop doing.
