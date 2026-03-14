Route QA-passed work to the next phase or finalize to content handoff.

Load the `kd-handoff-dev` skill and follow its complete workflow.

1. Find tickets with `to: dev`, `status: done`, QA verdict PASS or PASS-WITH-NOTES, and `qa_full_gate: passed`
2. Require `review_status: passed` before routing
3. Check phase routing: if more phases remain → create next-phase ticket for dev
4. If final phase → create content handoff ticket in `_handoff/queue/`
5. For multi-phase specs: build cumulative change summary from ALL phase archives
6. Update spec status to `implemented`
7. Archive the completed dev handoff ticket

Ticket to finalize: $ARGUMENTS
