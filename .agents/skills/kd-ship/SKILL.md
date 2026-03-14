---
name: kd-ship
description: "Finalization hygiene checks before kd-handoff-dev. Validates QA Full Gate, review gate, changelog/content readiness, and operational notes. Does not deploy. Triggers on: ship prep, finalization prep, pre-finalization checks."
---

# kd-ship — Finalization Hygiene

Prepare a dev-complete ticket for safe `/kd-handoff-dev` finalization.

## Workflow

### Step 1: Pick Ticket
1. Find ticket with `to: dev` and `status: done`
2. Read QA report, review report, and dev implementation logs

### Step 2: Run Readiness Checks
Required checks:
- `qa_full_gate: passed`
- `review_status: passed` on originating dev ticket
- Rollback plan present and concrete
- Env/config/migration notes present when applicable
- Completion summary is cumulative for multi-phase specs

### Step 3: Output
Append:

```markdown
## Ship Readiness Report
- **Agent**: ship
- **Date**: {ISO date}

| Check | Status | Notes |
|-------|--------|-------|
| QA Full Gate passed | ✅/❌ | {detail} |
| Review gate passed | ✅/❌ | {detail} |
| Rollback plan present | ✅/❌ | {detail} |
| Config/migration notes complete | ✅/❌ | {detail} |
```

If all pass: keep ticket ready for `/kd-handoff-dev`.
If any fail: set `status: blocked` and list required fixes.

## Rules
- Never run deploy commands
- Fail-fast on missing full-gate QA evidence
