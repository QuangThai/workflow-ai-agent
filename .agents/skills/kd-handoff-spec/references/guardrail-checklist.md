# Spec Guardrail Checklist

Run these checks against the candidate draft spec before approving it. The goal is to catch ambiguity here rather than passing it downstream to dev/QA, where it becomes churn and wasted cycles.

## Core Requirements

- [ ] Has Problem section (≥50 words, describes user pain with concrete scenarios)
- [ ] Has Technical Approach with Backend AND/OR Frontend subsections
- [ ] Has ≥3 Acceptance Criteria (testable, specific, with expected outcomes)
- [ ] Has Phase breakdown (required if effort ≥ M)
- [ ] No Open Questions marked as "blocker" or unresolved
- [ ] Effort estimate is proportional to scope (S: <5 files, M: 5-15 files, L: 15+ files)
- [ ] Fact Ledger present with no unresolved "Facts to Look Up"
- [ ] Phase tasks use strict hierarchical numbering (`1.1`, `1.2`, `2.1`, ...)
- [ ] Every phase task has at least one matching acceptance criterion

## Design Completeness

- [ ] Current State Analysis exists with concrete code/file references
- [ ] Solution Options Matrix exists with ≥2 options compared (chosen + rejected with reasons)
- [ ] Impact Analysis exists with affected services, consumers, breakage risks, and mitigations
- [ ] Includes measurable success metric(s) with baseline and target
- [ ] Includes explicit service scope (which repos/services are touched)
- [ ] Includes rollback strategy and risk level (`low|medium|high`)

## Conditional Sections (include when applicable)

- [ ] If API changes: API Contract section present with endpoints, request/response schemas, error codes
- [ ] If DB changes: Data Model & Migration section present with schema, migration plan, rollback
- [ ] If UI changes: UX/UI Flow section present with state matrix (loading/empty/error/success)
- [ ] If config changes: Environment & Configuration section present with env vars, flags
- [ ] Testing Strategy covers unit + integration + E2E/manual scenarios
- [ ] Rollout & Observability plan exists with rollback trigger and steps (required for risk ≥ medium)

## Quality

- [ ] No placeholder text remains (`TBD`, `TODO`, `...`, `How to solve it`, empty bullets)
- [ ] No low-confidence assumptions in critical-path sections without mitigation plan
- [ ] Out of Scope section exists (prevents scope creep during dev)

## Semantic Sanity Check

Evaluate the spec against these three questions. They catch ambiguity that structural checks miss:

1. "Is this spec actionable for a developer with NO prior context?" — Reject if it contains ambiguous requirements like "improve performance" without specific metrics, or "refactor" without listing target files.
2. "Could QA test every acceptance criterion without inventing expected behavior?" — Reject if criteria lack specific expected outcomes.
3. "Could release ship this without guessing env vars, migration order, or rollback commands?" — Reject if operational details are missing for risk ≥ medium.
