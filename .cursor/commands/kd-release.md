Verify release status and update product state for shipped work.

Load the `kd-release` skill and follow its complete workflow.

This agent does NOT deploy code — deployment happens outside this workflow.

1. Pick up release tickets from `_handoff/queue/` where `to: release`
2. Ask the user whether the change is live to its intended audience
3. If not yet deployed: present readiness checklist, STOP, and wait for user to deploy
4. If live: run post-release verification (health checks, smoke tests)
5. Update `_context/product-state.md` and spec status to `released`
6. Create content handoff ticket for changelog/blog generation

Ticket to release: $ARGUMENTS
