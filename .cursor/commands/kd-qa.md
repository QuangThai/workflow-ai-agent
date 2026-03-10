Run QA verification on completed dev work.

1. Load the `kd-qa` skill
2. Follow the skill's full workflow (Step 1 → Step 7)
3. Find tickets where `to: dev` and `status: done`
4. Run automated checks (lint, typecheck, tests) using Fast Gate or Full Gate
5. Perform manual review checklist on changed files
6. Verify acceptance criteria for the current phase
7. Generate QA Report with verdict: PASS / FAIL / PASS-WITH-NOTES
8. Fill out Progress Ledger to detect loops and stalls
9. Route result: PASS → suggest `/kd-handoff-dev`, FAIL → return to dev with actionable feedback

Ticket to QA: $ARGUMENTS
