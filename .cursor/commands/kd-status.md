Show the current state of the kd-* development pipeline.

1. Scan `_handoff/queue/` for all pending tickets
2. Group by status: `pending`, `in-progress`, `done`
3. Group by target agent: `dev`, `qa`, `release`, `content`
4. Sort by priority (P0 > P1 > P2)
5. Flag any ticket with `loop_count >= 2` as at-risk
6. Check `_context/specs/` for specs in each lifecycle status (draft, approved, implemented, released, archived)
7. Present a summary table like:

```
📋 Pipeline Status
═══════════════════

🔴 Blocked (loop_count >= 3):
  (none or list)

🟡 In Progress:
  HO-005 | P1 | dev | Phase 2/3 | "Feature X"

🟢 Pending:
  HO-006 | P2 | qa  | Phase 1/1 | "Bug fix Y"

📊 Specs: 2 draft, 1 approved, 1 implemented, 3 released
```
