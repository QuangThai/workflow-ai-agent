---
name: kd-handoff-dev
description: "Route QA-passed dev work: advance to next implementation phase or finalize to content handoff. Use after QA passes with PASS or PASS-WITH-NOTES verdict. Triggers on: handoff dev, finalize, route after QA, advance phase, next phase."
---

# kd-handoff-dev — Dev Finalization (or Next Phase)

Route completed, QA-passed work to its next destination:
- Next dev phase (for multi-phase specs), or
- Content handoff (default completion path)

This stage is the final technical gate before the feature is considered complete in the workflow.

## Workflow

### Step 1: Pick Up QA-Passed Work
1. Review `_context/lessons.md` for relevant patterns
2. Find eligible tickets in `_handoff/queue/`:
   - `to: dev` AND `status: done`
   - Latest QA verdict is `PASS` or `PASS-WITH-NOTES`
   - `qa_full_gate: passed`
   - `review_status: passed`
3. If multiple tickets match: sort by priority and ask the user which to route
4. Read ticket + referenced spec and determine `current_phase` / `total_phases`

Fail-fast:
- If no eligible tickets exist, STOP with: `No QA-passed work ready for finalization.`
- If spec is missing, STOP and report dependency.

### Step 2: Phase Routing

#### Path A: More phases remain (`current_phase < total_phases`)
1. Determine next HO-ID
2. Create next-phase ticket (`to: dev`, `status: pending`, `current_phase: next`)
3. Reset `loop_count: 0` for the new phase
4. Archive completed phase ticket
5. Print: `📋 Phase {current_phase}/{total_phases} complete. Next phase queued for dev.`
6. STOP

#### Path B: Final phase complete (`current_phase == total_phases`)
Continue to Step 3.

### Step 3: Finalize Docs + Spec
1. Verify relevant docs are updated where required (service `AGENTS.md` / `PRD.md`)
2. Update spec status: `approved` -> `implemented`

### Step 4: Create Content Handoff (Default Completion Path)
Determine next HO-ID and create a content ticket:

```markdown
---
id: HO-{next_id}
from: dev
to: content
priority: {priority}
status: pending
created: {ISO timestamp}
spec: SPEC-XXX
total_phases: 1
current_phase: 1
loop_count: 0
output_mode: last_message
announcement_scope: internal
qa_full_gate: passed
evidence_paths: [{paths from QA/review/browser evidence}]
---
```

Content handoff must include:
- What was completed (user-visible + technical summary)
- QA evidence + test summary
- Key files changed
- Known limitations/follow-ups

### Step 5: Archive Completed Dev Handoff
Archive the finalized dev handoff ticket (`status: done` then move to `_handoff/archive/`).

### Step 6: Complete
```text
✅ Finalization complete: _handoff/queue/{content-ticket}
📋 Spec: SPEC-XXX (implemented)
📝 Next: Run /kd-content
```

## Rules
- Require `qa_full_gate: passed` and `review_status: passed` before routing
- Only route current finalized phase; never skip unfinished phases
- Keep cumulative summaries for multi-phase specs
- Archive finalized ticket immediately after creating next ticket
