# _handoff — Inter-Agent Queue

Handoff files pass work between pipeline stages.
Each file is a structured markdown document with metadata and an explicit contract.

## Flow
```
queue/          → Agent picks up → Processes → Moves to archive/
```

## Naming Convention
```
HO-XXX-{from}-to-{to}-{title}.md
```

Example: `HO-001-brainstorm-to-dev-transcript-upload-validation.md`

## Contract Chain

Every handoff is a **contract** between the producing and receiving agent. The producing agent validates its own output before handing off — never pass incomplete work downstream.

Each handoff MUST include:
- **task_description** — What the receiving agent should do (with ALL relevant context, no ambiguity)
- **acceptance_criteria** — What "done" looks like (testable, specific)
- **context_keys** — Which `_context/` artifacts the receiving agent must read
- **output_mode** — How much history to pass:
  - `full_history` — Use for backward handoffs (QA → dev) where the receiver needs to see what was tested
  - `last_message` — Use for forward handoffs (release → content) where only the final result matters

## Output Mode Guide

| Direction | Example | Mode | Rationale |
|-----------|---------|------|-----------|
| Forward (happy path) | brainstorm → dev | `full_history` | Dev needs full spec context |
| Forward (late stage) | release → content | `last_message` | Content only needs release notes |
| Backward (failure) | QA → dev | `full_history` | Dev must see what QA tested and why it failed |
| Backward (re-spec) | dev → brainstorm | `last_message` | Brainstorm needs the problem, not implementation details |

## Handoff Template
```markdown
---
id: HO-XXX
from: brainstorm|dev|qa|release|content
to: dev|release|content
priority: P0|P1|P2
status: pending|in-progress|done|blocked
created: YYYY-MM-DDTHH:MM:SSZ
spec: SPEC-XXX (if applicable)
total_phases: {N — total implementation phases, default 1 for single-phase work}
current_phase: {N — which phase this handoff covers, default 1}
loop_count: {N — QA→dev cycle count, starts at 0}
output_mode: full_history|last_message
service_scope: {list of affected services/repos}
risk_level: low|medium|high
rollback_plan: {short rollback strategy}
started_at: YYYY-MM-DDTHH:MM:SSZ (optional)
qa_started_at: YYYY-MM-DDTHH:MM:SSZ (optional)
qa_completed_at: YYYY-MM-DDTHH:MM:SSZ (optional)
released_at: YYYY-MM-DDTHH:MM:SSZ (optional)
---

# Title

## Contract
- **task_description**: {What the receiving agent should do — complete, unambiguous}
- **acceptance_criteria**: {What "done" looks like — testable}
- **context_keys**: {List of `_context/` files to read}
- **output_mode**: full_history|last_message

## Context
What and why.

## Deliverables
- [ ] Checklist of what needs to be done

## Acceptance Criteria
- [ ] How to verify it's done

## Implementation Plan (Phased)
### Phase 1: {Name}
- Tasks:
  - [ ] **1.1** {task}
  - [ ] **1.2** {task}

### Phase 2: {Name}
- Tasks:
  - [ ] **2.1** {task}
  - [ ] **2.2** {task}

## References
- Links to _context/ docs, PRD sections, etc.
```

## Rules
- Never create a handoff without a Contract section
- Producing agent validates own output before writing the handoff
- Backward handoffs (FAIL routing) MUST include specific, actionable instructions — not just "fix it"
- Archive completed handoffs immediately to keep the queue clean
- **Fail-fast on missing inputs**: If a required artifact (spec, ticket, config file) is missing or has malformed frontmatter, STOP immediately. Set ticket `status: blocked`, report the exact missing dependency, and do not guess forward.
- Track optional lifecycle timestamps when available to compute cycle-time and QA turnaround metrics.
- For phased work, implementation tasks should use hierarchical numbering (`1.1`, `1.2`, `2.1`, ...).

## ID Allocation
To determine the next handoff ID:
1. Scan all files in both `_handoff/queue/` and `_handoff/archive/`
2. Parse the numeric prefix from filenames (e.g., `HO-012-...` → 12)
3. Next ID = max found + 1, zero-padded to 3 digits (e.g., `HO-013`)
4. Never reuse IDs, even for archived tickets
5. Filename format: `HO-{NNN}-{from}-to-{to}-{slug}.md`

## Ticket Lifecycle
Each ticket follows a linear state machine. Only one active ticket per spec+phase at any time.

### State transitions
```
pending → in-progress → done → archived
                      ↘ blocked (escalate to user)
```

### Stage routing (who updates `to:`)
| Transition | Who updates | New `to` value | Notes |
|------------|-------------|----------------|-------|
| handoff-spec creates ticket | kd-handoff-spec | `dev` | Initial creation |
| Dev picks up | kd-dev | (no change) | Status: `pending` → `in-progress` |
| Dev completes | kd-dev | (no change) | Status: `in-progress` → `done` |
| QA picks up done ticket | kd-qa | (no change) | QA reads tickets where `to: dev` AND `status: done` |
| QA PASS / PASS-WITH-NOTES | kd-qa | (no change) | Appends QA Report with verdict PASS or PASS-WITH-NOTES |
| QA FAIL | kd-qa | (no change) | Appends QA Report, resets `status: pending`, increments `loop_count` |
| handoff-dev routes | kd-handoff-dev | `release` (or creates next phase ticket) | Archives completed ticket |
| Release completes | kd-release | `content` (new ticket) | Archives release ticket |
| Content completes | kd-content | (no change) | Archives content ticket |
| Release deploy fails | kd-release | `dev` (new ticket) | Rollback + feedback ticket with failure details |
| Content needs repo changes | kd-content | `dev` (new ticket) | Docs/code updates that bypass dev/QA |

### QA fail reuse rule
On QA failure, do NOT create a new ticket. Instead:
1. Append QA Report and Progress Ledger to the **existing** ticket
2. Reset `status: pending` (so dev picks it up again)
3. Increment `loop_count`
4. The dev agent reads the appended QA feedback on next pickup

### PASS-WITH-NOTES definition
Verdict `PASS-WITH-NOTES` means QA passed but has cosmetic or non-blocking observations. Rules:
- Only for observations that do NOT require code changes or acceptance criteria corrections
- If any note requires a code fix, the verdict must be `FAIL`
- Notes are informational for the release agent (e.g., "consider renaming this variable in a future cleanup")
