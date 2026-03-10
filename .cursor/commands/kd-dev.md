Pick up the highest-priority pending handoff ticket and implement it.

1. Load the `kd-dev` skill
2. Follow the skill's full workflow (Step 1 → Step 7)
3. Check `_handoff/queue/` for pending tickets where `to: dev`
4. Read the Contract section as the primary instruction set
5. Implement changes following service-specific conventions from AGENTS.md and PRD.md
6. Self-verify with lint, typecheck, and tests before marking done
7. Run the Elegance Gate for non-trivial changes
8. Update the handoff ticket with implementation log
9. If a specific HO-ID is given, work on that ticket directly

Ticket or task: $ARGUMENTS
