---
name: kd-dev
description: "Implement features, fix bugs, and address QA feedback for approved dev handoff tickets. Handles greenfield scaffolding, multi-service changes, phased specs, QA loop fixes, and direct bugfixes. Use whenever code needs to be written or changed. Triggers on: implement, code, dev, fix, patch, bugfix, resume work, continue phase, address QA feedback, scaffold, wire up."
---

# kd-dev — Development Agent

Pick up approved handoff tickets and implement features in the current project codebase. This is the agent that writes code — it reads specs, understands the plan, and turns it into working software.

## Core Principle

**Implement the current phase only.** Multi-phase specs break work into independently testable slices. Focus on the current phase's tasks and acceptance criteria. Do not silently advance phases or implement future-phase work.

---

## Workflow

### Bug Fix Fast Path
When invoked directly for a bug (no existing handoff ticket), create a minimal ticket before implementing:

1. Scan `_handoff/queue/` and `_handoff/archive/` to determine next HO-ID
2. Create a minimal handoff ticket in `_handoff/queue/`:

```markdown
---
id: HO-{next_id}
from: dev
to: dev
priority: {P0|P1|P2 based on severity}
status: in-progress
created: {ISO timestamp}
spec: none
total_phases: 1
current_phase: 1
loop_count: 0
output_mode: full_history
---

# Bugfix: {Brief Description}

## Contract
- **task_description**: {What the bug is and how to fix it}
- **acceptance_criteria**: {How to verify the bug is fixed}
- **context_keys**: {relevant files}
- **output_mode**: full_history

## Bug Details
- **Symptoms**: {What the user reported or what's failing}
- **Root cause**: {To be filled after investigation}
- **Files affected**: {To be filled during implementation}
```

3. Then proceed to Step 2 (Load Context) with this self-created ticket.
4. If investigation reveals a deeper architectural issue, STOP and escalate to the full Feature Pipeline (`/kd-brainstorm`).

### Step 1: Pick Up Work
1. List `_handoff/queue/` for pending tickets where `to: dev`
2. Sort by priority (P0 > P1 > P2)
3. If multiple candidates: present them and ask the user which to work on
4. Read the full handoff ticket
5. Identify current phase: `current_phase` of `total_phases` from frontmatter
6. **Fail-fast**: If no pending tickets found and no bug reported → STOP and report. If a ticket references a missing spec → STOP and report.

### Step 2: Load Context
1. Review `_context/lessons.md` for patterns relevant to this task
2. Read the **Contract** section — this is the primary instruction set
3. Read all files listed in `context_keys`
4. Read the referenced spec from `_context/specs/` — focus on the **current phase** section
5. If `context_keys` references research notes in `_context/research/`, read them — they affect library choices, API semantics, and integration patterns
6. Read relevant `AGENTS.md` and `PRD.md` for code conventions in affected services
7. Scan affected files in the current phase's scope using `finder` / `Read`
8. If `loop_count > 0`: Read previous QA report and Progress Ledger — the `instruction_or_question` field tells you exactly what to fix. Address that **first**.

> **Bug Fix Fast Path**: For tickets where `spec: none`, skip spec/PRD reading (steps 4-6) but still read `AGENTS.md` for code conventions in the affected service.

### Step 2.5: Pre-Implementation Gate

Scale the gate to the size and risk of the change:

**For bugfixes and QA loop fixes (`loop_count > 0`):**
- Bug reproduced or symptom clearly identified?
- Expected correct behavior known?
- Likely fix area identified?
- Regression test plan clear?

**For small/single-service changes:**
- Acceptance criteria clear enough to implement?
- Affected files and services identified?
- Test strategy clear?
- Any unresolved product ambiguity?

**For multi-service, contract/schema, or M/L effort changes — also check:**
- Contract owner: which service defines the canonical behavior?
- Backward compatibility: can services be deployed independently?
- Deploy order: does one service need to go first?
- Schema/event changes: are they backward-compatible?
- Cross-service test plan clear?

**Decision rules for ambiguity:**
- **Self-solve**: implementation details, code patterns, naming, test structure, local refactors, which existing utility to use
- **Ask the user**: product behavior, ambiguous UX, conflicting acceptance criteria, missing credentials/access, multiple valid approaches with different user-visible outcomes
- **Route back** to `/kd-handoff-spec` or `/kd-brainstorm`: major plan mismatch, missing API contracts, architectural disagreement with spec

Do not guess on product behavior, API contracts, schema semantics, or user-visible outcomes. Self-resolve implementation mechanics using existing code, research, and conventions.

### Step 3: Implement

Follow the current phase's task list as a structured implementation plan:

#### 3.1: Map tasks to files
- Read the current phase's tasks from the spec/handoff (e.g., tasks 1.1, 1.2, 1.3...)
- List files to create or modify for each task
- List tests to add or update
- Note config, migration, or documentation changes

#### 3.2: Implement in dependency order
- Build foundation layers first (contracts, providers, data models)
- Then consumers (controllers, pages, UI components)
- Write tests alongside implementation, not as an afterthought
- For multi-service changes: maintain deploy compatibility between services

#### 3.3: Work incrementally
- Make a small change → verify it works → continue
- Run narrow checks after each logical slice (e.g., typecheck a new module before building on it)
- Avoid large speculative diffs — prefer validated increments

#### 3.4: Handle special cases

**If `loop_count > 0` (QA feedback loop):**
- Fix the specific issues from QA's `instruction_or_question` first
- Add or adjust tests for the failing behavior
- Track what changed since last QA in the implementation log

**If greenfield (files/services don't exist yet):**
- Scaffold using the framework's CLI or project structure conventions
- Missing files are expected, not a blocker
- Follow the spec's prescribed structure

**If the spec doesn't match reality:**
- Minor factual mismatch (e.g., different file path): update the handoff ticket with a note, continue implementing
- Major plan/business mismatch: STOP and route back to `/kd-handoff-spec`

**If React/frontend code is involved:**
Load the `vercel-react-best-practices` skill for performance patterns (memoization, code splitting, data fetching). Apply its guidelines to changed components.

#### 3.5: Mark progress
As you complete each task, mark it done in the implementation log:
```
- [x] 1.1 Scaffold NestJS project
- [x] 1.2 Install dependencies
- [ ] 1.3 Create JWT strategy (in progress)
```

### Step 4: Self-Verify

Run checks before marking done. Scale verification to the size of the change:

**One check suite** (e.g., single full-stack app, or small fix): Run checks inline in the main context.

**Two or more check suites** (e.g., separate FE + BE — whether in one monorepo or multiple repos): Spawn one Task per check suite in a **single message** so they run concurrently.

**What to check (dev Fast Gate):**

| Check | Command (adapt per service) |
|-------|----------------------------|
| Lint | `eslint` / `ruff check` / service-specific linter |
| Format | `prettier --check` / `ruff format --check` |
| Types | `tsc --noEmit` / `mypy` / `pyright` |
| Tests | Run changed/related tests (not full suite) |

Each check should produce a structured result:

| Check | Status | Details |
|-------|--------|---------|
| Lint | ✅/❌ | {output summary} |
| Types | ✅/❌ | {error count} |
| Tests | ✅/❌ | {pass/total, failing names} |

**When to run Full Gate (full test suite + build):**
- Final phase of a multi-phase spec
- Schema/contract/migration changes
- Risky changes flagged in the handoff

**If checks fail:** Fix the issues. If a subagent Task fails (timeout, crash), retry that service's checks inline. Do not skip checks for any changed service.

**If React files changed:** Run `vercel-react-best-practices` audit on changed components.

### Step 5: Elegance Gate
Before marking done, pause and self-review:
1. Ask: *"Would a staff engineer approve this diff?"*
2. For non-trivial changes: *"Is there a more elegant way?"*
3. If the fix feels hacky: re-implement the clean solution — but scope this to the current change. Don't refactor unrelated code.
4. Skip this for simple fixes (single-line changes, config updates, QA loop fixes addressing specific feedback).

### Step 6: Update Handoff
Update the handoff ticket: `status: pending` → `status: done`

Add the implementation log to the ticket:

```markdown
## Implementation Log
- Phase: {current_phase} of {total_phases}
- Files changed:
  - {file path} — {brief description of change}
- Tests added:
  - {test file} — {N tests} ({what they cover})
- Loop iteration: {loop_count}

### Phase Task Status
- [x] {task_id} {task description}
- [x] {task_id} {task description}
- [x] {task_id} {task description}

### Change Summary Since Last QA (if loop_count > 0)
- {what changed specifically to address prior QA feedback}

### Acceptance Criteria Evidence
| Criterion | Evidence | Verified By |
|-----------|----------|-------------|
| {criterion from current phase} | {file/test/command} | {test name or check} |

### API Changes
- Endpoints added/changed: {list — or "None"}
- Auth/permission changes: {summary — or "None"}

### Config Changes
- Env vars added: {list with defaults — or "None"}
- Feature flags: {flag name, state — or "None"}

### Data Changes
- Migration: {yes/no — migration name}
- Rollback: {reversible? how}

### Services Touched This Phase
- {service list — only services changed in this phase}

### Verification Commands
```bash
{exact commands QA should run to verify — copy from what you actually ran}
```

### Known Limitations / Follow-ups
- {anything not covered, deferred to later phase}
- {edge cases, notes for QA or release}

### Spec Deviations
- {any differences discovered between spec and reality — or "None"}
```

### Step 7: Complete

**If implementation succeeded:**
```
✅ Implementation done: {title} (Phase {current_phase}/{total_phases})
📁 Files changed: {count}
🧪 Tests: {pass/fail summary}
⏭️ Next: Run /kd-qa to verify
```

**If blocked (cannot complete):**
Save all progress made so far, then:
1. Update the handoff ticket with partial implementation log (mark completed vs pending tasks)
2. Set `status: blocked`
3. Classify the blocker:
   - **spec** — ambiguous/incorrect spec → route to `/kd-handoff-spec`
   - **access** — missing credentials, env vars, services → ask user
   - **dependency** — external service/library issue → ask user
   - **architecture** — fundamental design issue → route to `/kd-brainstorm`
4. Record the blocker, what was tried, and the recommended next action

```
🚧 Blocked: {title} (Phase {current_phase}/{total_phases})
📋 Blocker: {classification} — {short description}
📁 Progress saved: {what was completed}
⏭️ Next: {recommended action}
```

---

## Rules
- **Current phase only** — implement only the current phase's tasks and acceptance criteria
- **Read before writing** — always read existing code before modifying it
- **Smallest reasonable diffs** — don't rewrite files to change a few lines
- **Follow existing patterns** — don't introduce new libraries without checking the project uses them
- **Write tests** for new functionality — tests alongside code, not as an afterthought
- **Never skip checks** — always run lint/types/tests before marking done
- **Save progress when blocked** — never discard partial work. Update the ticket with what's done and what's stuck
- **Self-solve implementation, ask on product** — figure out code patterns yourself, but ask the user when product behavior is ambiguous
- **Write to `_context/lessons.md`** when: (a) you discover a non-obvious root cause during debugging, (b) a QA loop reveals a recurring pattern, or (c) you find an undocumented codebase convention
