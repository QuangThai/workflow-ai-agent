Convert an approved brainstorm spec into a dev-ready handoff ticket.

Load the `kd-handoff-spec` skill and follow its complete workflow.

1. Read the candidate spec from `_context/specs/` — must have `status: draft`
2. Run guardrail validation using `references/guardrail-checklist.md`
3. Decompose into phases with strict task numbering (1.1, 1.2, 2.1, ...)
4. Update spec status to `approved`
5. Create handoff ticket in `_handoff/queue/` with full Contract section
6. Update `_context/product-state.md` with the active spec

If no spec ID is given, auto-detect the latest draft spec.

Spec to hand off: $ARGUMENTS
