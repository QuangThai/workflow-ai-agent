Pick up the highest-priority pending handoff ticket and implement it.

Load the `kd-dev` skill and follow its complete workflow.

1. Scan `_handoff/queue/` for pending tickets where `to: dev`, sorted by priority
2. Read the Contract section — this is the primary instruction set
3. Load context: spec, research notes, `_context/lessons.md`, service AGENTS.md
4. Implement the current phase only — do not advance phases
5. Self-verify: lint, typecheck, and relevant tests
6. Run the Elegance Gate for non-trivial changes
7. Update the handoff ticket with implementation log and set `status: done`

If a specific HO-ID is provided, work on that ticket directly.

Ticket or task: $ARGUMENTS
