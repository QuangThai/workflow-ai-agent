Execute the release process for QA-passed, handoff-ready work.

1. Load the `kd-release` skill
2. Follow the skill's full workflow (Step 1 → Step 7)
3. Pick up release tickets from `_handoff/queue/` where `to: release`
4. Run pre-deploy checklist — all service checks must pass (non-mutating)
5. Present deploy command to user — NEVER auto-execute deploy
6. After user confirms: run post-deploy verification and health checks
7. Update `_context/product-state.md` and spec status to `released`
8. Create content handoff for changelog/blog generation

Ticket to release: $ARGUMENTS
