Quickly fix a bug using the Bug Fix Fast Path (skip brainstorm/spec).

1. Load the `kd-dev` skill and use the Bug Fix Fast Path workflow
2. Create a minimal handoff ticket in `_handoff/queue/` with bug details
3. Investigate the bug: find root cause and identify affected files
4. Implement the fix with minimal diff
5. Self-verify: run lint, typecheck, and relevant tests
6. Update the handoff ticket with implementation log
7. If investigation reveals deeper architectural issues, STOP and escalate — suggest `/kd-brainstorm`
8. Print summary and suggest running `/kd-qa`

Bug to fix: $ARGUMENTS
