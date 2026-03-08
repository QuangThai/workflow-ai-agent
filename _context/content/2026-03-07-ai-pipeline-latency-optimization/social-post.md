<!-- [agent: content] Generated from SPEC-002 -->

## LinkedIn / X Post

🚀 **Shipped: AI pipeline latency optimization for Scopelytics AI**

We profiled our meeting analysis pipeline and found 5 places where we were doing redundant work — counting the same tokens 6-7 times, triggering unnecessary LLM recall passes, and re-classifying transcripts we'd already seen.

The fix? No new models, no new infra. Just:
→ Batch token counting (6 calls → 1)
→ Smarter recall threshold (50% → 30%)
→ Halved retry budget (5 max LLM calls → 3)
→ Redis-cached classification
→ Eliminated double counting

Sometimes the fastest AI is the one that stops repeating itself.

#AI #LLM #PerformanceOptimization #OpenAI #FastAPI
