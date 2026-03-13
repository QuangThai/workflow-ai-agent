---
name: kd-handoff-spec
description: "Convert brainstorm specs into dev-ready handoff tickets with phased implementation plans. Use after brainstorm produces a draft spec and the user approves it. Triggers on: handoff spec, approve spec, queue for dev, spec to dev, create handoff, spec ready."
---

# kd-handoff-spec — Spec → Dev Handoff

The bridge between brainstorm and dev. This skill validates a draft spec, decomposes it into implementation phases, and creates a structured handoff ticket that dev agents can pick up without asking clarifying questions.

Why this step matters: ambiguity that passes through here becomes dev churn and QA loops downstream. Every hour spent validating the spec saves multiple hours of rework later.

---

## Workflow

### Step 1: Validate Spec

1. Review `_context/lessons.md` for patterns relevant to spec validation
2. Read the candidate spec from `_context/specs/SPEC-XXX-*.md`
3. Confirm the spec has `status: draft` — this skill validates drafts and promotes them to `approved`
4. Verify it has the essential sections: Problem, Solution/Proposed Solution, Technical Approach, Acceptance Criteria
5. Scan `_handoff/queue/` and `_handoff/archive/` filenames to determine the next HO-ID (see `_handoff/README.md` § ID Allocation)
6. If incomplete, list what's missing and ask the user to resolve

**Fail-fast**: If the spec file is missing, has no Problem section, or has `status` other than `draft`, STOP and report the issue. A spec that's already `approved` or `implemented` doesn't need this step — it's already been through the gate.

### Step 2: Guardrail Validation

Run quality checks before proceeding. The goal is to ensure the spec is actionable for dev, testable for QA, and operationally complete for release.

Read `references/guardrail-checklist.md` and evaluate the spec against every applicable check. Skip conditional sections that don't apply (e.g., skip the API Contract check if there are no API changes).

**If any check fails:** Return specific failure reasons with the exact missing information. Do not proceed until resolved. Do NOT "invent certainty" — if phase decomposition would require guessing because the spec is vague, fail back to brainstorm rather than silently converting ambiguity into tasks.

**If the user pushes back on a guardrail:** Use judgment. Some specs (effort S, low risk, single service) genuinely don't need every section. The three semantic questions at the bottom of the checklist are the real bar — if a developer could implement, QA could test, and release could ship without guessing, the spec is ready.

### Step 3: Phase Decomposition

Break the spec into implementation phases. Required for effort ≥ M, recommended for effort S.

Decompose based on:
1. **Dependency order** — Phase N must not depend on Phase N+1
2. **Risk mitigation** — High-risk / high-uncertainty items first
3. **Incremental value** — Each phase delivers independently testable value
4. **Effort balance** — Each phase fits within ~1-2 agent sessions
5. **Rollback independence** — Reverting Phase N does not break Phase N-1

Each phase must specify:
- Scope (exact files affected)
- Acceptance criteria (testable per-phase)
- Dependencies on prior phases
- Estimated effort (S/M within the phase)
- Numbered task list using `N.M` format (phase.task)

**Task numbering**: Phase 1 tasks are `1.1`, `1.2`, etc. Phase 2 tasks are `2.1`, `2.2`, etc. This consistency matters because dev and QA agents reference tasks by these IDs.

**Single-phase normalization**: Even S-sized work gets a Phase 1 so downstream agents can always rely on `current_phase` and per-phase criteria.

**Persist phases to spec**: After decomposition, update the spec file in `_context/specs/` with the finalized phase breakdown. The spec is the durable source of truth; the handoff ticket references it.

### Step 4: Update Spec Status

Edit the spec file — change `status: draft` → `status: approved`

### Step 5: Create Handoff Ticket

Create a handoff file in `_handoff/queue/` following `_handoff/README.md` naming convention (`HO-{NNN}-{from}-to-{to}-{slug}.md`).

Read `references/dev-ticket-template.md` for the full template. Key points:

- The **Contract** section scopes to the **current phase** (Phase 1), not the entire spec. Dev agents are phase-aware and treat the Contract as their primary instruction set.
- Include `service_scope`, `risk_level`, and `rollback_plan` in frontmatter (these help downstream agents).
- Group scope by affected service/repo rather than hardcoded "Service A / Service B" labels.
- Reference the spec for full context; the ticket is a focused action plan, not a spec copy.

### Step 6: Update Product State

Append to `_context/product-state.md`:
- Add spec to "Active Specs" section
- Add decision summary to "Recent Decisions" section

### Step 7: Notify

```
✅ Handoff created: _handoff/queue/{filename}
📋 Spec: _context/specs/SPEC-XXX-*.md (approved)
🎯 Priority: {priority}
📦 Phases: {total_phases}
⏭️ Next: Run /kd-dev to start implementation
```

---

## Rules

- Validate drafts, then approve — never create a handoff from an unvalidated spec
- Block ambiguity here rather than passing it to dev/QA — this is the last quality gate before code gets written
- Scope the handoff Contract to the current phase's acceptance criteria, with the full spec linked in references
- Follow `_handoff/README.md` for IDs, filenames, and required frontmatter fields
- Keep the ticket project-agnostic — group by affected repos/services, don't hardcode service names
- **Write to `_context/lessons.md`** when: (a) repeated validation failures expose a missing spec pattern, (b) a guardrail check consistently catches the same gap, or (c) the user corrects an assumption about the approval process
