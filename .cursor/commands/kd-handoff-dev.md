Route QA-passed work to the next phase or prepare release handoff.

Load the `kd-handoff-dev` skill and follow its complete workflow.

1. Find tickets with `to: dev`, `status: done`, and QA verdict PASS or PASS-WITH-NOTES (Full Gate required)
2. Check phase routing: if more phases remain → create next-phase ticket for dev
3. If final phase → create release handoff ticket in `_handoff/queue/`
4. For multi-phase specs: build cumulative change summary from ALL phase archives
5. Update spec status to `implemented`
6. Archive the completed dev handoff ticket

Ticket to finalize: $ARGUMENTS
