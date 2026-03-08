# CLAUDE.md — Local Agent SOP Entry Point

This workspace uses a file-based, local-first pipeline with shared context, structured handoff contracts, and phased execution.

## Pipeline

`/kd-brainstorm -> /kd-handoff-spec -> /kd-dev -> /kd-qa -> /kd-handoff-dev -> /kd-release -> /kd-content`

## Source of Truth

- Orchestrator rules: `AGENTS.md`
- Operational guide: `README.md`
- Shared context: `_context/`
- Work queue: `_handoff/queue/`
- Handoff contract rules: `_handoff/README.md`
- Lessons learned: `_context/lessons.md`

---

## Agent Behavior Rules

These rules apply to ALL agents at ALL times, regardless of pipeline stage.

### 1. Plan Before You Build
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions).
- If something goes sideways, **STOP and re-plan immediately** — never keep pushing a failing approach.
- Use plan mode for verification steps, not just implementation.
- Write detailed specs upfront to reduce ambiguity.

### 2. Subagent Strategy
- Use subagents liberally to keep the main context window clean.
- Offload research, exploration, and parallel analysis to subagents.
- For complex problems, throw more compute at it via subagents.
- One task per subagent for focused execution.

### 3. Self-Improvement Loop
- After ANY correction from the user: update `_context/lessons.md` with the pattern.
- Write rules for yourself that prevent the same mistake from recurring.
- Ruthlessly iterate on these lessons until the mistake rate drops.
- Review `_context/lessons.md` at session start for the relevant project.

### 4. Verification Before Done
- Never mark a task complete without proving it works.
- Diff behavior between main and your changes when relevant.
- Ask yourself: *"Would a staff engineer approve this?"*
- Run tests, check logs, demonstrate correctness.

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask *"Is there a more elegant way?"*
- If a fix feels hacky: *"Knowing everything I know now, implement the elegant solution."*
- Skip this for simple, obvious fixes — do not over-engineer.
- Challenge your own work before presenting it.

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Do not ask for hand-holding.
- Point at logs, errors, failing tests — then resolve them.
- Zero context switching required from the user.
- Go fix failing CI tests without being told how.

### 7. Simplicity & Minimal Impact
- Make every change as simple as possible. Touch minimal code.
- Find root causes. No temporary fixes. Senior developer standards.
- Changes should only touch what is necessary. Avoid introducing bugs.

---

## Pipeline Modes

### Feature Pipeline (default)
Full pipeline for new features, improvements, and non-trivial changes:
```
/kd-brainstorm → /kd-handoff-spec → /kd-dev → /kd-qa → /kd-handoff-dev → /kd-release → /kd-content
```

### Bug Fix Fast Path
For bug reports and failing tests — skip brainstorm and spec, go straight to fix:
```
Bug report → /kd-dev → /kd-qa → /kd-handoff-dev → /kd-release
```
Rules for fast path:
- Only for clear bugs with reproducible symptoms (failing tests, error logs, user-reported defects).
- Dev agent creates a minimal handoff ticket directly (no spec required).
- Must still pass full QA gates — no shortcuts on verification.
- If the bug reveals a deeper architectural issue, escalate to the full Feature Pipeline.

---

## Core Principles

1. **Fact-first** — Gather and verify facts before brainstorming solutions
2. **Phased execution** — Break work into independently testable phases (required for effort ≥ M)
3. **Contract-driven handoffs** — Every handoff includes task_description, acceptance_criteria, context_keys, and output_mode
4. **Guardrailed specs** — Automated validation before specs enter the dev pipeline
5. **Loop detection** — Progress Ledger prevents infinite QA→dev cycles (escalate at loop_count ≥ 3)
6. **Self-improvement** — Every correction becomes a lesson; every lesson prevents future mistakes

---

## Stage Rules

### 1) `/kd-brainstorm`
- Must read `_context/product-state.md` and related specs/decisions first.
- Must review `_context/lessons.md` for relevant past mistakes.
- Must build a **Fact Ledger** (Known Facts → Unknown / Needs Research → Assumptions) before proposing solutions.
- Must treat research as **mandatory** (never optional), and resolve all critical unknowns before drafting the spec.
- Must dispatch parallel research **subagents** (Track A: codebase feasibility, Track B: best-practice/docs, Track C: external/competitive patterns when applicable) to keep main context clean.
- Must produce:
  - Draft spec in `_context/specs/SPEC-XXX-{slug}.md`
  - Research note in `_context/research/YYYY-MM-DD-{topic}.md` (sources + distilled best practices)

### 2) `/kd-handoff-spec`
- Only runs on an approved draft.
- Must pass **Guardrail Validation** (problem depth, acceptance criteria count, phase breakdown, no unresolved blockers).
- Must produce **Phase Decomposition** for effort ≥ M (dependency order, risk-first, incremental value, rollback independence).
- Updates spec status to `approved`.
- Creates handoff ticket with **Contract section**: `_handoff/queue/HO-XXX-brainstorm-to-dev-{slug}.md`.

### 3) `/kd-dev`
- Picks highest-priority `to: dev` ticket.
- Reviews `_context/lessons.md` for patterns relevant to this task.
- Reads the **Contract** section first — this is the primary instruction set.
- Implements the **current phase only** (from `current_phase` in frontmatter).
- If `loop_count > 0`: Must address the specific `instruction_or_question` from the QA Progress Ledger.
- Before marking done: self-review — *"Would a staff engineer approve this? Is there a more elegant way?"*
- Updates ticket status, phase tracking, and implementation log.

### 4) `/kd-qa`
- Runs required checks (lint/types/tests/contracts).
- Verifies acceptance criteria with evidence (per-phase criteria).
- Fills out **Progress Ledger** (is_satisfied, is_in_loop, is_progress_being_made, loop_count).
- Routes PASS forward or FAIL back to dev with **concrete, actionable instructions**.
- **Loop detection**: Escalates to user at `loop_count >= 3`.

### 5) `/kd-handoff-dev`
- Requires QA PASS.
- Updates spec to `implemented`.
- Creates release ticket and archives completed dev ticket.

### 6) `/kd-release`
- Never auto-deploy; present command and wait for user confirmation.
- Verifies post-deploy health/smoke checks.
- Updates spec to `released` and creates content ticket.

### 7) `/kd-content`
- Generates changelog/blog/social/docs artifacts.
- Saves artifacts to `_context/content/YYYY-MM-DD-{slug}/`.
- Archives content ticket and marks spec `archived`.

---

## File Standards

### Spec frontmatter

```yaml
---
id: SPEC-XXX
title: Feature Title
status: draft|approved|implemented|released|archived
priority: P0|P1|P2
effort: S|M|L
created: YYYY-MM-DD
author: agent:brainstorm|dev|qa|release|content
---
```

### Handoff frontmatter

```yaml
---
id: HO-XXX
from: brainstorm|dev|qa|release
to: dev|qa|release|content
priority: P0|P1|P2
status: pending|in-progress|done|blocked
created: YYYY-MM-DDTHH:MM:SSZ
spec: SPEC-XXX
total_phases: N
current_phase: N
loop_count: 0
output_mode: full_history|last_message
---
```

## Quality Gates

- Backend: `ruff check`, `ruff format --check`, `mypy`, `pytest`
- Frontend: `npm run lint`, `npm run build`, `npm run check:api-contract`
