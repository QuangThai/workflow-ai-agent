Quickly fix a bug using the Bug Fix Fast Path — skip brainstorm and spec stages.

Load the `kd-dev` skill and use the Bug Fix Fast Path workflow.

1. Create a minimal handoff ticket in `_handoff/queue/` with bug details
2. Investigate the bug: find root cause and identify affected files
3. Implement the fix with minimal, focused diff
4. Self-verify: run lint, typecheck, and relevant tests
5. Update the handoff ticket with implementation log
6. If investigation reveals deeper architectural issues → STOP and suggest `/kd-brainstorm`

Next step after fix: run `/kd-qa` to verify.

Bug to fix: $ARGUMENTS
