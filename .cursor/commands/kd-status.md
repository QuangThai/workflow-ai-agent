Show the current state of the development pipeline.

1. Scan `_handoff/queue/` for all tickets
2. Group by status: `pending`, `in-progress`, `done`, `blocked`
3. Identify work stage for each ticket:
   - `to: dev` + `status: pending` → Awaiting dev
   - `to: dev` + `status: in-progress` → Dev in progress
   - `to: dev` + `status: done` → Ready for QA
   - `to: content` → Awaiting content/final completion
4. Sort by priority (P0 > P1 > P2)
5. Flag any ticket with `loop_count >= 2` as at-risk
6. Flag tickets missing finalization gates (`review_status != passed` or `qa_full_gate != passed`) as finalization-blocked
7. Scan `_context/specs/` for spec lifecycle counts (draft, approved, implemented, archived)

Present a summary like:

```
📋 Pipeline Status
═══════════════════

🔴 Blocked (loop_count >= 3):
  (none)

🟡 In Progress:
  HO-005 | P1 | dev | Phase 2/3 | "Feature X"

🟢 Pending:
  HO-006 | P2 | ready for QA | Phase 1/1 | "Bug fix Y"

📊 Specs: 2 draft, 1 approved, 1 implemented
```
