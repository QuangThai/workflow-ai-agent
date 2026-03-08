---
name: kd-brainstorm
description: "Brainstorm product ideas, research solutions, and draft specs. Use when starting new features, exploring approaches, or running product discovery sessions. Triggers on: brainstorm, ideate, explore, research."
---

# kd-brainstorm — Product Discovery & Research

Brainstorm ideas, research solutions, analyze competitors, and produce draft specs for the current project.

## Workflow

### Step 1: Load Context
Read these files to ground the session in current project state:

1. `_context/lessons.md` — scan for patterns relevant to this topic
2. `_context/product-state.md` — current priorities and active specs
3. `_context/decisions/` — past architecture and product decisions (if any exist)
4. `_context/research/` — past research notes (check for related prior research to avoid duplication)
5. `_context/specs/` — scan existing spec filenames to determine the **next SPEC ID** (e.g., if `SPEC-005` exists, next is `SPEC-006`)
6. Relevant `PRD.md` files (for impacted services) for requirements alignment
7. `_handoff/queue/` — any feedback from QA or dev that needs addressing

### Step 2: Fact Ledger
Before brainstorming solutions, gather and classify facts. This prevents hallucinated assumptions and ensures the solution is grounded in reality.

Create a structured ledger:

```markdown
## Fact Ledger

### Given / Verified Facts
- {Known truths from codebase, PRD, product-state, existing specs}
- {Confirmed constraints — infrastructure, budget, timeline}

### Facts to Look Up
- {Best practices for this problem domain — with target sources}
- {Library/framework capabilities — versions, limitations}
- {Competitor approaches — how others solved this}

### Facts to Derive
- {Performance impact estimates — based on current metrics}
- {Effort breakdown — files affected, migration complexity}
- {Dependency analysis — what breaks if we change X}

### Educated Guesses
- {Hypotheses requiring validation — "Redis can handle N req/s"}
- {Assumptions about user behavior or system load}
- {Confidence level: High / Medium / Low}
```

Complete the ledger with what you know from Step 1. The "Facts to Look Up" items become the research questions for Step 3.

### Step 3: Research (required for non-trivial features)
Research across three tracks using **subagents (Task tool)** to run tracks in parallel and keep the main context clean.

For trivial features (effort S, no unknowns in the Fact Ledger), skip to Step 4.

#### Execution Strategy
Spawn **3 Task calls in a single assistant message** (one per track). This runs them in parallel as independent subagents. Each subagent does all the heavy research internally and returns only a short summary — the raw search results, page contents, and intermediate steps stay inside the subagent's context and never enter the main context window.

**Task prompt template** — each Task call must include:
1. The research goal and specific questions to answer (from "Facts to Look Up")
2. The tools to use (listed per track below)
3. The Fact Ledger items relevant to that track (copy them in)
4. This exact output instruction: *"Return ONLY a structured summary (max 500 words). Do not include raw search results, full page contents, or intermediate reasoning. Format: bullet points grouped by finding category."*

**Source quality policy (required):**
- Tier 1 (preferred): official specifications and official product/framework documentation
- Tier 2 (secondary): vendor/community articles, engineering blogs
- Any Tier 2 claim must be corroborated by at least one Tier 1 source before it is treated as "Verified Fact"

#### Track A — Technical Feasibility & Documentation
Research goal: Can we build this with our current stack?

Tools the subagent should use (in priority order):
1. `finder` — Check existing codebase for related implementations and patterns
2. `mcp__ref__ref_search_documentation` — Search official docs (include library name and feature)
3. `mcp__ref__ref_read_url` — Read full content of relevant doc pages found
4. `mcp__context7__query_docs` — Secondary source for framework-specific code examples

Expected summary sections: (1) relevant existing patterns found, (2) official documentation guidance with links, (3) library capabilities/limitations, (4) integration risks, (5) recommended approach with file paths.

#### Track B — External Best Practices
Research goal: How have others solved this? What are the proven patterns?

Tools the subagent should use (in priority order):
1. `mcp__ref__ref_search_documentation` — Authoritative library docs first
2. `mcp__exa__web_search_exa` — Implementation patterns, blog posts, recent references
3. `librarian` — Cross-repo codebase understanding for open-source patterns
4. `web_search` — Broader technical approaches (fallback)

Expected summary sections: (1) top 5 sources with links, (2) 5-10 distilled best practices, (3) what approaches were rejected and why, (4) clear recommendation with trade-offs.

#### Track C — Security & Compliance (if applicable)
Skip this track if the feature has no security implications. When skipping, spawn only 2 Tasks (Track A + B).

Research goal: What could go wrong? What must we protect against?

Tools the subagent should use:
1. `mcp__ref__ref_search_documentation` — Official security docs for relevant frameworks
2. `mcp__exa__web_search_exa` — OWASP guidelines, security advisories, known vulnerabilities
3. `finder` — Review existing security patterns in the codebase (search for auth, validation, sanitization patterns)

Expected summary sections: (1) applicable security standards, (2) potential vulnerabilities, (3) required mitigations, (4) compliance checklist.

#### Synthesis
After all subagents return their summaries (each ≤500 words), the main agent:
1. Combines the summaries — resolve conflicts between sources
2. Updates the Fact Ledger — promote guesses to verified facts, flag invalidated assumptions
3. Produces a recommendation with confidence level
4. Checks: are there still unresolved "Facts to Look Up"? If yes, do a targeted follow-up (single tool call, not another subagent round)

Document findings in `_context/research/YYYY-MM-DD-{topic}.md`:
- Top sources consulted (with links)
- 5-10 distilled best practices relevant to this project
- Clear recommendation with trade-offs
- What was rejected and why (if applicable)
- Updated Fact Ledger entries
- Source metadata per key finding: `source_url`, `publish_or_last_updated`, `retrieved_at`, `confidence`

### Step 4: Brainstorm Solutions
Based on verified facts and research findings, explore:
- **Problem definition** — What user pain does this solve? (grounded in Fact Ledger)
- **Solution options** — At least 2-3 approaches with trade-offs
- **Technical feasibility** — Validated against current project architecture
- **Scope estimation** — Small / Medium / Large effort (backed by dependency analysis)
- **Risk assessment** — What could go wrong? (informed by research Track C)

### Step 5: Draft Output
Create a **draft spec** in `_context/specs/` using the next available SPEC ID (determined in Step 1):

```markdown
---
id: SPEC-{next_id}
title: Feature Title
status: draft
priority: P0|P1|P2
effort: S|M|L
created: YYYY-MM-DD
author: agent:brainstorm
---

# SPEC-{next_id}: Feature Title

## Problem
What problem does this solve? (≥50 words, describe user pain)

## Fact Ledger
### Verified Facts
- {Key facts established during research}

### Assumptions
- {Educated guesses with confidence levels}

## Proposed Solution
How to solve it.

## Technical Approach
### Backend Changes
- Endpoints, models, services affected

### Frontend Changes
- Components, routes, API calls affected

## Phase Planning Rules
- Use numbered phases: `Phase 1`, `Phase 2`, ...
- Inside each phase, tasks MUST use hierarchical numbering:
  - Phase 1 tasks: `1.1`, `1.2`, `1.3`, ...
  - Phase 2 tasks: `2.1`, `2.2`, `2.3`, ...
- Keep each task atomic and testable in one dev iteration.
- Every task must map to at least one acceptance criterion.
- If effort is `M` or `L`, phase breakdown is mandatory.

## Phases (required for effort ≥ M)
### Phase 1: {Name}
- Objective: {what this phase delivers}
- Scope: {exact files/services}
- Tasks:
  - [ ] **1.1** {atomic task}
  - [ ] **1.2** {atomic task}
  - [ ] **1.3** {atomic task}
- Acceptance Criteria:
  - [ ] {testable criterion linked to 1.x tasks}
- Dependencies: none
- Risks: {known risks in this phase}
- Rollback: {how to revert this phase safely}

### Phase 2: {Name}
- Objective: {what this phase delivers}
- Scope: {exact files/services}
- Tasks:
  - [ ] **2.1** {atomic task}
  - [ ] **2.2** {atomic task}
- Acceptance Criteria:
  - [ ] {testable criterion linked to 2.x tasks}
- Dependencies: Phase 1
- Risks: {known risks in this phase}
- Rollback: {how to revert this phase safely}

## Acceptance Criteria
- [ ] Testable criterion 1
- [ ] Testable criterion 2
- [ ] Testable criterion 3

## Open Questions
- Things to resolve before dev (mark as "blocker" or "nice-to-have")
```

Before presenting the draft, run this **Spec Readiness Check**:
1. Each phase has clear objective/scope/dependencies.
2. Tasks are numbered by phase (`1.x`, `2.x`, ...).
3. Acceptance criteria are testable and mapped to tasks.
4. Rollback is defined per phase (or explicitly "not needed" with reason).
5. No blocker open questions remain.

### Step 6: User Review
Present the draft to the user for review:
- Summarize key decisions and trade-offs
- Highlight open questions
- Ask for approval or iteration

**On approval**: User says "approve" or runs `/kd-handoff-spec` to move to dev pipeline.
**On iteration**: Refine based on feedback and re-present.

**IMPORTANT**: Do NOT proceed to `/kd-handoff-spec` automatically. Always wait for explicit user approval.

## Rules
- Always check existing code before proposing new patterns
- Prefer extending existing architecture over introducing new dependencies
- Reference PRD sections when applicable
- Tag all outputs with `[agent: brainstorm]`
- Never skip the Fact Ledger — even for small features, fill in at least "Given / Verified Facts"
- If a research subagent fails or returns empty results, note the gap in the spec's "Open Questions" section rather than guessing
