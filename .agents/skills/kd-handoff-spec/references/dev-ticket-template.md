# Dev Handoff Ticket Template

Use this template when creating the initial handoff ticket in `_handoff/queue/`.
Adapt sections to what the spec actually touches — omit sections that don't apply rather than filling them with "None" everywhere.

```markdown
---
id: HO-{next_id}
from: brainstorm
to: dev
priority: {from spec}
status: pending
created: {ISO timestamp}
spec: SPEC-XXX
total_phases: {N}
current_phase: 1
loop_count: 0
service_scope: {list of affected services/repos}
risk_level: {low|medium|high — from spec}
rollback_plan: {short rollback strategy — from spec}
output_mode: full_history
---

# {Spec Title}

## Contract
- **task_description**: {What the dev agent should do for Phase 1 — full context, no ambiguity}
- **acceptance_criteria**: {Phase 1 acceptance criteria — copied from spec, refined for testability}
- **context_keys**: {Which `_context/` artifacts to read — specs, research, decisions}
- **output_mode**: full_history

## Context
{Brief summary of the problem and approved solution}

## Scope
{Group by affected service/repo — list endpoints, models, components, routes as relevant}

## Implementation Plan (Phased)

### Phase 1: {Phase Name} — {priority}
- **Scope**: {exact files to modify}
- **Objective**: {what this phase delivers}
- **Tasks**:
  - [ ] **1.1** {Step-by-step implementation task}
  - [ ] **1.2** {Step-by-step implementation task}
- **Acceptance Criteria**:
  - [ ] {Testable criterion mapped to 1.x tasks}
- **Depends on**: none
- **Rollback**: {how to revert this phase}

### Phase 2: {Phase Name} — {priority}
- **Scope**: {exact files to modify}
- **Objective**: {what this phase delivers}
- **Tasks**:
  - [ ] **2.1** {Step-by-step implementation task}
  - [ ] **2.2** {Step-by-step implementation task}
- **Acceptance Criteria**:
  - [ ] {Testable criterion mapped to 2.x tasks}
- **Depends on**: Phase 1
- **Rollback**: {how to revert this phase}

## Deliverables
- [ ] Implementation per phase
- [ ] Tests (unit + integration, per phase)
- [ ] Documentation updates if new endpoints/features

## Dev Notes
- Relevant files: {list key files to modify}
- Related tests: {list test files}
- Migration needed: {yes/no}

## Operational Context
- API changes: {summary from spec's API Contract — or "None"}
- Data/migration changes: {summary from spec's Data Model — or "None"}
- Config/env changes: {summary from spec's Environment section — or "None"}
- Feature flags: {flag names and defaults — or "None"}
- Testing strategy: {summary from spec's Testing Strategy}
- Rollout plan: {summary from spec's Rollout & Observability}
- Rollback: {rollback steps from spec}
- Deploy order: {if multi-service — which service deploys first}

## References
- Spec: `_context/specs/SPEC-XXX-*.md`
- Research: `_context/research/YYYY-MM-DD-*.md` (if applicable)
- Product docs: {relevant PRD or AGENTS.md files for affected services}
```
