---
name: kd-brainstorm
description: "Brainstorm product ideas, research solutions, and draft specs. Use when starting new features, exploring approaches, or running product discovery sessions. Triggers on: brainstorm, ideate, explore, research."
---

# kd-brainstorm — Product Discovery & Research

Brainstorm ideas, research solutions, analyze architecture, and produce implementation-ready specs. The goal is a spec so complete that developers can implement without asking clarifying questions.

## Core Principle

**Gather all the information first, design second.**

A spec that reaches dev with missing API contracts, undefined UI states, or unclear migration plans will cause QA loops. Invest time upfront in completeness.

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
8. Relevant `AGENTS.md` files — for code conventions, quality gates, and tech stack

### Step 2: Fact Ledger
Before brainstorming solutions, gather and classify facts. This prevents hallucinated assumptions and ensures the solution is grounded in reality.

Create a structured ledger:

```markdown
## Fact Ledger

### Given / Verified Facts
- {Known truths from codebase, PRD, product-state, existing specs}
- {Confirmed constraints — infrastructure, budget, timeline}
- {Tech stack and version details from AGENTS.md / PRD.md}

### Facts to Look Up
- {Best practices for this problem domain — with target sources}
- {Library/framework capabilities — versions, limitations}
- {Competitor approaches — how others solved this}
- {Current codebase architecture for affected areas}

### Facts to Derive
- {Performance impact estimates — based on current metrics}
- {Effort breakdown — files affected, migration complexity}
- {Dependency analysis — what breaks if we change X}
- {Data model implications — schema changes needed}

### Educated Guesses
- {Hypotheses requiring validation — "Redis can handle N req/s"}
- {Assumptions about user behavior or system load}
- {Confidence level: High / Medium / Low}
```

Complete the ledger with what you know from Step 1. The "Facts to Look Up" items become the research questions for Step 3.

### Step 3: Research (mandatory for effort ≥ M, recommended for S)

Research across three tracks using **subagents (Task tool)** to run tracks in parallel and keep the main context clean.

For trivial features (effort S, zero unknowns in the Fact Ledger), skip to Step 4 — but still fill in the Current State Analysis.

#### Execution Strategy
Spawn **3 Task calls in a single assistant message** (one per track). Each subagent does all the heavy research internally and returns only a short summary.

**Task prompt template** — each Task call must include:
1. The research goal and specific questions to answer (from "Facts to Look Up")
2. The tools to use (listed per track below)
3. The Fact Ledger items relevant to that track (copy them in)
4. This exact output instruction: *"Return ONLY a structured summary (max 500 words). Do not include raw search results, full page contents, or intermediate reasoning. Format: bullet points grouped by finding category."*

**Source quality policy (required):**
- Tier 1 (preferred): official specifications and official product/framework documentation
- Tier 2 (secondary): vendor/community articles, engineering blogs
- Any Tier 2 claim must be corroborated by at least one Tier 1 source before it is treated as "Verified Fact"

#### Track A — Technical Feasibility & Codebase Analysis
Research goal: Can we build this with our current stack? What existing patterns should we reuse?

Tools the subagent should use (in priority order):
1. `finder` — Check existing codebase for related implementations, patterns, and architecture
2. `Read` — Read key files to understand current architecture deeply
3. `mcp__context7__query_docs` — Framework-specific code examples and up-to-date docs
4. `web_search` / `mcp__exa__web_search_exa` — Official docs, recent references

Expected summary sections: (1) current architecture map for affected areas, (2) existing patterns to reuse with file paths, (3) library capabilities/limitations, (4) integration risks and constraints, (5) recommended approach with specific files to modify.

#### Track B — External Best Practices
Research goal: How have others solved this? What are the proven patterns?

Tools the subagent should use (in priority order):
1. `mcp__context7__query_docs` — Up-to-date library docs and examples
2. `mcp__exa__web_search_exa` — Implementation patterns, blog posts, recent references
3. `librarian` — Cross-repo codebase understanding for open-source patterns
4. `web_search` — Broader technical approaches (fallback)

Expected summary sections: (1) top 5 sources with links, (2) 5-10 distilled best practices, (3) what approaches were rejected and why, (4) clear recommendation with trade-offs.

#### Track C — Security & Compliance (if applicable)
Skip this track if the feature has no security implications. When skipping, spawn only 2 Tasks (Track A + B).

Research goal: What could go wrong? What must we protect against?

Tools the subagent should use:
1. `mcp__context7__query_docs` — Official security docs for relevant frameworks
2. `mcp__exa__web_search_exa` — OWASP guidelines, security advisories, known vulnerabilities
3. `finder` — Review existing security patterns in the codebase

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

### Step 4: Design the Solution

This is the core design step. It must be thorough enough that a developer can implement from the spec alone.

#### 4.1 Current State Analysis
Map the existing architecture for the affected areas. This is mandatory for all non-trivial work.

- What services/modules are involved?
- What is the current request/data flow?
- What existing patterns should be reused?
- What constraints/invariants must not break?
- List specific code references: `path/to/file` — what it currently does

#### 4.2 Solution Options Matrix
Explore at least 2-3 approaches and compare them explicitly:

| Option | Summary | Reuses Existing? | Pros | Cons | Risk | Effort | Confidence |
|--------|---------|-------------------|------|------|------|--------|------------|
| A | ... | yes/no | ... | ... | low/med/high | S/M/L | High/Med/Low |
| B | ... | yes/no | ... | ... | low/med/high | S/M/L | High/Med/Low |
| C | ... | yes/no | ... | ... | low/med/high | S/M/L | High/Med/Low |

Select one and document:
- **Chosen**: {option} — why it was selected
- **Rejected**: {options} — why they were rejected
- **Non-goals / Out of scope**: what this feature explicitly does NOT cover

#### 4.3 Impact & Dependency Analysis
Analyze what will be affected by the change:

| Area | Current Behavior | Planned Change | Breakage Risk | Mitigation |
|------|------------------|----------------|---------------|------------|
| {service/module} | {current} | {planned} | low/med/high | {strategy} |

Also determine:
- Affected services/repos
- Affected files/modules (specific paths)
- Downstream consumers that depend on changed behavior
- Backward compatibility notes
- Deployment ordering constraints (if multi-service)

#### 4.4 Detailed Technical Design
Structure the technical approach based on what the feature touches.

**Backend Changes:**
- Endpoints, models, services affected
- Request/response flow
- Auth/permission changes
- Error handling approach

**Frontend Changes:**
- Components, routes, hooks affected
- Data fetching strategy
- State management approach

**Conditional design sections** — read `references/spec-sections.md` for the detailed template of each:
- If API changes → include **API Contract** (endpoints, request/response, errors, examples)
- If DB changes → include **Data Model & Migration** (schema, migration plan, backfill, rollback)
- If UI changes → include **UX/UI Flow** (routes, states, validation, copy, accessibility)
- If config changes → include **Environment & Configuration** (env vars, flags, secrets, infra)

If a section is not applicable, write `N/A — {reason}`. Do not silently omit.

#### 4.5 Testing Strategy
Define how this feature will be tested:

- **Unit tests**: {what to test, key scenarios}
- **Integration tests**: {API contracts, service interactions}
- **E2E / Manual scenarios**: {critical user flows to verify}
- **Regression risks**: {existing functionality that might break}
- **Test data / fixtures**: {what test data is needed}

#### 4.6 Rollout & Observability
Define how this feature will be shipped safely:

- **Release strategy**: {big bang / feature flag / canary / phased}
- **Feature flags**: {flag names, default values, rollout plan}
- **Metrics to watch**: {latency, error rate, usage}
- **Alerts / dashboards**: {what to monitor post-deploy}
- **Smoke tests**: {specific post-deploy checks}
- **Rollback trigger**: {when to rollback — specific conditions}
- **Rollback steps**: {exact commands/actions to revert}

#### 4.7 Downstream Readiness Sweep
Before drafting the spec, verify:
- [ ] Can dev implement this without asking clarifying questions?
- [ ] Can QA test this without inventing expected behavior?
- [ ] Can release ship this without guessing env vars/migration order?
- [ ] Are all acceptance criteria testable with specific expected outcomes?

If any answer is "no", go back and fill in the missing details.

### Step 5: Draft Spec
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
What problem does this solve? (≥50 words, describe user pain — concrete scenarios, not abstract)

## Fact Ledger
### Verified Facts
- {Key facts established during research — with source references}

### Assumptions
- {Educated guesses with confidence levels — High/Medium/Low}

## Current State Analysis
### Existing Architecture
- {Services/modules involved}
- {Current request/data flow}
- {Existing patterns to reuse}
- {Constraints/invariants that must not break}

### Relevant Code References
- `{path/to/fileA}` — {what it currently does}
- `{path/to/fileB}` — {integration point}

## Solution Options
| Option | Summary | Reuses Existing? | Pros | Cons | Risk | Effort | Confidence |
|--------|---------|-------------------|------|------|------|--------|------------|
| A | ... | ... | ... | ... | ... | ... | ... |
| B | ... | ... | ... | ... | ... | ... | ... |

## Proposed Solution
- **Chosen**: {option and why}
- **Rejected**: {options and why}

## Out of Scope
- {Explicit non-goals — what this feature does NOT cover}

## Impact Analysis
| Area | Current Behavior | Planned Change | Breakage Risk | Mitigation |
|------|------------------|----------------|---------------|------------|
| ... | ... | ... | ... | ... |

## Technical Approach
### Backend Changes
- {Specific endpoints, models, services to modify/create}
- {Request/response flow description}

### Frontend Changes
- {Specific components, routes, hooks to modify/create}
- {State management approach}

### API Contract (if applicable — see references/spec-sections.md)
{Endpoint definitions, request/response schemas, error codes, examples}

### Data Model & Migration (if applicable — see references/spec-sections.md)
{Schema changes, migration plan, backfill strategy, rollback}

### UX/UI Flow (if applicable — see references/spec-sections.md)
{Routes, state matrix, validation, copy, accessibility}

### Environment & Configuration (if applicable — see references/spec-sections.md)
{Env vars, feature flags, secrets, infra dependencies}

## Service Scope
- **Services affected**: {list repos/services touched}
- **Risk level**: low|medium|high
- **Rollback strategy**: {how to revert if things go wrong}

## Success Metrics
| Metric | Baseline | Target |
|--------|----------|--------|
| {metric} | {current value} | {target value} |

## Testing Strategy
- **Unit tests**: {key scenarios}
- **Integration tests**: {API/service interaction tests}
- **E2E / Manual**: {critical user flows}
- **Regression risks**: {existing features to verify}
- **Test data needed**: {fixtures, seeds, mocks}

## Rollout & Observability
- **Release strategy**: {feature flag / canary / big bang}
- **Metrics to watch**: {what to monitor}
- **Smoke tests**: {post-deploy verification}
- **Rollback trigger**: {specific conditions}
- **Rollback steps**: {exact revert actions}

## Phases (required for effort ≥ M)
### Phase 1: {Name}
- Objective: {what this phase delivers}
- Scope: {exact files/services}
- Tasks:
  - [ ] **1.1** {atomic task}
  - [ ] **1.2** {atomic task}
  - [ ] **1.3** {atomic task}
- Acceptance Criteria:
  - [ ] {testable criterion linked to 1.x tasks — with specific expected outcome}
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
  - [ ] {testable criterion linked to 2.x tasks — with specific expected outcome}
- Dependencies: Phase 1
- Risks: {known risks in this phase}
- Rollback: {how to revert this phase safely}

## Acceptance Criteria (overall)
- [ ] {Testable criterion with specific expected outcome}
- [ ] {Testable criterion with specific expected outcome}
- [ ] {Testable criterion with specific expected outcome}

## Open Questions
- {Mark as "blocker" or "nice-to-have" — include owner/assignee and target date}
```

### Spec Readiness Check
Before presenting the draft, verify ALL of these:
1. Each phase has clear objective/scope/dependencies
2. Tasks are numbered by phase (`1.x`, `2.x`, ...)
3. Acceptance criteria are testable and mapped to tasks, with specific expected outcomes
4. Rollback is defined per phase (or explicitly "not needed" with reason)
5. No blocker open questions remain
6. Service Scope is defined with risk level and rollback strategy
7. At least one measurable success metric with baseline and target
8. Current State Analysis exists with concrete code references
9. Solution Options Matrix exists with at least 2 options compared
10. Impact Analysis covers affected services/consumers/compatibility
11. Conditional sections present when applicable (API Contract, Data Model, UX/UI Flow, Environment)
12. Testing Strategy covers unit + integration + E2E
13. Rollout & Observability plan exists with rollback trigger and steps
14. No placeholder text remains (`TBD`, `TODO`, `...`, empty bullets)

### Step 6: User Review
Present the draft to the user for review:
- Summarize key decisions and trade-offs (chosen vs. rejected options)
- Highlight the impact analysis — what will be affected
- Call out any remaining open questions
- Show the downstream readiness sweep results
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
- Never skip Current State Analysis for non-trivial features
- If a research subagent fails or returns empty results, note the gap in the spec's "Open Questions" section rather than guessing
- If a conditional section is not applicable, write `N/A — {reason}` rather than omitting it silently
- **Write to `_context/lessons.md`** when: (a) research reveals a common mistake pattern, (b) a design approach was rejected for non-obvious reasons, or (c) the user corrects a spec assumption
