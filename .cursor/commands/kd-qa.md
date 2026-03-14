Run QA verification on completed dev work.

Load the `kd-qa` skill and follow its complete workflow.

1. Find tickets where `to: dev` and `status: done` — these are dev-completed, QA-ready
2. Run automated checks via parallel subagents (one per impacted service): lint, typecheck, tests
3. Use gate mode policy: `fast` default, `full` required before `/kd-handoff-dev`
4. Run `/kd-browser-qa` for UI/web changes and attach evidence paths
5. Perform manual review checklist on changed files
6. Verify acceptance criteria for the current phase with evidence
7. Generate QA Report with verdict: PASS | FAIL | PASS-WITH-NOTES
8. Fill out Progress Ledger to detect loops and stalls
9. Route: PASS → suggest `/kd-handoff-dev` (only with Full Gate) | FAIL → return ticket to dev with actionable feedback

Ticket to QA: $ARGUMENTS
