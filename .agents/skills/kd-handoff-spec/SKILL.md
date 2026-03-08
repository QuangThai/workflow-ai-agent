---
name: kd-handoff-spec
description: "Convert approved brainstorm specs into dev-ready handoff tickets. Use after brainstorm approval to queue work for dev agents. Triggers on: handoff spec, approve spec, queue for dev."
---

# kd-handoff-spec — Spec → Dev Handoff

Convert approved specs from brainstorm into actionable dev handoff tickets.

## Workflow

### Step 1: Validate Spec
1. Read the approved spec from `_context/specs/SPEC-XXX-*.md`
2. Verify it has: Problem, Solution, Technical Approach, Acceptance Criteria
3. If incomplete, list what's missing and ask user to resolve

### Step 2: Update Spec Status
Edit the spec file — change `status: draft` → `status: approved`

### Step 3: Create Handoff Ticket
Create a handoff file in `_handoff/queue/`:

```markdown
---
id: HO-XXX
from: brainstorm
to: dev
priority: {from spec}
status: pending
created: {ISO timestamp}
spec: SPEC-XXX
---

# {Spec Title}

## Context
{Brief summary of the problem and approved solution}

## Scope
- **Backend**: {list affected areas — endpoints, models, services, migrations}
- **Frontend**: {list affected areas — components, routes, hooks, API calls}
- **Shared**: {any _context updates needed}

## Implementation Plan
1. {Step-by-step implementation order}
2. {Dependencies between steps}
3. {What can be parallelized}

## Deliverables
- [ ] Backend implementation
- [ ] Frontend implementation
- [ ] Tests (unit + integration)
- [ ] PRD.md updates if new endpoints/features

## Acceptance Criteria
{Copied from spec, refined for testability}

## Dev Notes
- Relevant files: {list key files to modify}
- Related tests: {list test files}
- Migration needed: {yes/no}

## References
- Spec: `_context/specs/SPEC-XXX-*.md`
- PRD: `scopelytics-ai-backend/PRD.md` §{section}
- PRD: `scopelytics-ai-frontend/PRD.md` §{section}
```

### Step 4: Update Product State
Append to `_context/product-state.md`:
- Add spec to "Active Specs" section
- Add decision summary to "Recent Decisions" section

### Step 5: Notify
Print summary:
```
✅ Handoff created: _handoff/queue/{filename}
📋 Spec: _context/specs/SPEC-XXX-*.md (approved)
🎯 Priority: {priority}
⏭️ Next: Run /kd-dev to start implementation
```

## Rules
- Never create handoff without an approved spec
- Always include file paths for dev to modify
- Always reference PRD sections
