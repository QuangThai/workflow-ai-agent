---
name: kd-handoff-dev
description: "Route QA-passed dev work: advance to the next implementation phase or create the release handoff. Use after QA passes with PASS or PASS-WITH-NOTES verdict. Triggers on: handoff dev, prepare release, finalize, route after QA, advance phase, next phase, phase complete."
---

# kd-handoff-dev — Dev → Release Handoff (or Next Phase)

Route completed, QA-passed work to its next destination: either the next implementation phase (back to dev) or a release handoff. This is the traffic controller between QA and release.

Why this step matters: it ensures only fully verified work reaches release, multi-phase specs advance correctly one phase at a time, and the release agent receives a complete picture of what was built.

---

## Workflow

### Step 1: Pick Up QA-Passed Work

1. Review `_context/lessons.md` for relevant patterns
2. Find eligible tickets in `_handoff/queue/`:
   - `to: dev` AND `status: done`
   - Latest QA Report section has verdict `PASS` or `PASS-WITH-NOTES`
   - Latest QA Report used **Full Gate** (not just Fast Gate)
3. If multiple tickets match: sort by priority and ask the user which to route
4. Read the full handoff ticket
5. Read `current_phase` and `total_phases` from frontmatter

**Verdict handling:**
- `PASS`: proceed normally
- `PASS-WITH-NOTES`: proceed, but carry the notes forward to the release ticket's "PASS-WITH-NOTES observations" field for visibility
- `FAIL` or missing QA Report: redirect to `/kd-qa`
- Fast Gate only (no Full Gate): redirect to `/kd-qa` with instruction to run Full Gate before release

**Fail-fast**: If no QA-passed tickets with Full Gate are found, STOP and report "No QA-passed work ready for handoff." If the spec referenced by the ticket is missing, STOP and report.

### Step 2: Phase Routing

Read `current_phase` and `total_phases` from the handoff ticket frontmatter. This determines the path forward.

#### Path A: More phases remain (`current_phase < total_phases`)

1. Determine the next HO-ID (scan `_handoff/queue/` and `_handoff/archive/`)
2. Create a new handoff ticket in `_handoff/queue/` using `_handoff/README.md` naming convention:

```markdown
---
id: HO-{next_id}
from: qa
to: dev
priority: {carry from current ticket}
status: pending
created: {ISO timestamp}
spec: SPEC-XXX
total_phases: {total_phases}
current_phase: {next_phase}
loop_count: 0
origin_handoff_id: {ID of the original handoff from kd-handoff-spec}
service_scope: {carry from current ticket}
risk_level: {carry from current ticket}
output_mode: full_history
---

# {Spec Title} — Phase {next_phase}

## Contract
- **task_description**: Implement Phase {next_phase} of {Spec Title}. Read the spec for phase details, scope, and tasks. Prior phases are complete.
- **acceptance_criteria**: {Copy Phase {next_phase} acceptance criteria from spec}
- **context_keys**: {spec path, archived prior phase tickets, relevant _context/ files}
- **output_mode**: full_history

## Context
Phase {current_phase} completed and passed QA. This ticket covers Phase {next_phase}.

## Implementation Plan
### Phase {next_phase}: {Phase Name from spec}
- **Scope**: {from spec}
- **Objective**: {from spec}
- **Tasks**: {copy from spec}
- **Acceptance Criteria**: {copy from spec}
- **Depends on**: Phase {current_phase}

## References
- Spec: `_context/specs/SPEC-XXX-*.md`
- Prior phase archive: `_handoff/archive/HO-{current_id}-*.md`
```

3. Archive the completed phase's ticket: set `status: done`, then move/rename the file to `_handoff/archive/` following naming convention
4. Print: `📋 Phase {current_phase}/{total_phases} complete. Next phase queued for dev.`
5. **STOP** — do not proceed to release handoff.

#### Path B: Final phase complete (`current_phase == total_phases`)

Continue to Step 3.

### Step 3: Finalize Documentation

Before creating the release handoff, verify documentation is current:

1. Check if product docs (PRD.md, AGENTS.md) for affected services need updates based on what was implemented
2. Update spec status: `approved` → `implemented`

### Step 4: Create Release Handoff

**If `total_phases > 1`**: Before writing the ticket, read all archived phase tickets from `_handoff/archive/` for this spec. The release ticket must summarize changes from ALL phases, not just the final one. This is critical because the release agent needs the complete picture to verify the release.

Determine the next HO-ID, then create a release handoff in `_handoff/queue/` following `_handoff/README.md` naming convention.

Read `references/release-ticket-template.md` for the full template. Key points:

- The **Contract** tells the release agent to confirm release status (not to deploy) — the release agent asks the user whether the change is live
- Include `service_scope`, `risk_level`, and `rollback_plan` in frontmatter
- Carry forward PASS-WITH-NOTES observations from QA into the "Release Notes" section
- The "Changes" section must be cumulative across all phases for multi-phase specs
- The `origin_handoff_id` links back to the first handoff for this spec (for audit trail)

### Step 5: Archive Completed Handoff

Archive the final phase's dev handoff ticket: set `status: done`, then move/rename the file to `_handoff/archive/`.

### Step 6: Complete

```
✅ Release handoff ready: _handoff/queue/{filename}
📋 Spec: SPEC-XXX (implemented)
🚀 Next: Run /kd-release to verify release and hand off to content
```

---

## Rules

- Only route tickets whose latest QA result is PASS or PASS-WITH-NOTES with Full Gate — anything less must go back to QA first
- For multi-phase specs, build the cumulative change summary from ALL phase archives before creating the release ticket
- Archive explicitly: finalize metadata (`status: done`), then move/rename to `_handoff/archive/`
- Carry operational details forward exactly — don't make the release agent guess env vars, migration order, or rollback steps
- Follow `_handoff/README.md` for IDs, filenames, and required frontmatter fields
- Keep tickets project-agnostic — group by affected repos/services, don't hardcode service names
- When creating next-phase tickets, always reset `loop_count: 0` for the new phase
- **Write to `_context/lessons.md`** when: (a) release routing reveals a recurring pipeline gap, (b) multi-phase archiving exposes missing information, or (c) the same deployment assumption mismatch recurs
