---
name: kd-brainstorm
description: "Brainstorm product ideas, research solutions, and draft specs. Use when starting new features, exploring approaches, or running product discovery sessions. Triggers on: brainstorm, ideate, explore, research."
---

# kd-brainstorm — Product Discovery & Research

Brainstorm ideas, research solutions, analyze competitors, and produce draft specs for Scopelytics AI.

## Workflow

### Step 1: Load Context
1. Read `_context/product-state.md` for current priorities
2. Read `_context/decisions/` for past decisions
3. Read relevant `PRD.md` files (backend/frontend) for requirements alignment
4. Check `_handoff/queue/` for any feedback from QA or dev that needs addressing

### Step 2: Brainstorm
Based on the user's topic, explore:
- **Problem definition** — What user pain does this solve?
- **Solution options** — At least 2-3 approaches with trade-offs
- **Technical feasibility** — Check against current architecture (FastAPI backend, Next.js frontend)
- **Scope estimation** — Small / Medium / Large effort
- **Risk assessment** — What could go wrong?

### Step 3: Research (if needed)
Use available tools to research:
- Use `web_search` or `librarian` for technical approaches
- Use `finder` to check existing codebase for related implementations
- Use `mcp__context7__query_docs` for framework-specific guidance
- Document findings in `_context/research/YYYY-MM-DD-{topic}.md`

### Step 4: Draft Output
Create a **draft spec** in `_context/specs/`:

```markdown
---
id: SPEC-XXX
title: Feature Title
status: draft
priority: P0|P1|P2
effort: S|M|L
created: YYYY-MM-DD
author: agent:brainstorm
---

# SPEC-XXX: Feature Title

## Problem
What problem does this solve?

## Proposed Solution
How to solve it.

## Technical Approach
### Backend Changes
- Endpoints, models, services affected

### Frontend Changes
- Components, routes, API calls affected

## Acceptance Criteria
- [ ] Testable criteria

## Open Questions
- Things to resolve before dev
```

### Step 5: User Review
Present the draft to the user for review:
- Summarize key decisions and trade-offs
- Highlight open questions
- Ask for approval or iteration

**On approval**: User runs `/kd-handoff-spec` to move to dev pipeline.
**On iteration**: Refine and re-present.

## Rules
- Always check existing code before proposing new patterns
- Prefer extending existing architecture over introducing new dependencies
- Reference PRD sections when applicable
- Tag all outputs with `[agent: brainstorm]`
