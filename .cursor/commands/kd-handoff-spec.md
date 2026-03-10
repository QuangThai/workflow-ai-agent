Convert an approved brainstorm spec into a dev-ready handoff ticket.

1. Load the `kd-handoff-spec` skill
2. Follow the skill's full workflow (Step 1 → Step 7)
3. Validate the spec against all guardrail checks (programmatic + LLM)
4. Decompose into phases with strict task numbering (1.1, 1.2, 2.1, ...)
5. Create handoff ticket in `_handoff/queue/` with full Contract section
6. Update spec status to `approved` and update `_context/product-state.md`
7. If no spec ID is given, auto-detect the latest draft spec

Spec to convert: $ARGUMENTS
