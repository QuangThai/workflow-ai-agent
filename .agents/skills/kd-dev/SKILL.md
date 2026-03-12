---
name: kd-dev
description: "Pick up handoff tickets and implement features across backend and frontend. Use when ready to code approved specs. Triggers on: dev, implement, build, code."
---

# kd-dev — Development Agent

Pick up approved handoff tickets and implement features in the current project codebase.

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
3. Present to user: "Found {N} pending tickets. Starting with: {title} (P{X})"
4. Read the full handoff ticket
5. Identify current phase: `current_phase` of `total_phases` from frontmatter
6. **Fail-fast**: If no pending tickets are found and no bug was reported, STOP and report "No pending work in queue." If a ticket references a spec that doesn't exist, STOP and report the missing dependency.

### Step 2: Load Context
1. Review `_context/lessons.md` for patterns relevant to this task
2. Read the **Contract** section — this is the primary instruction set
3. Read all files listed in `context_keys`
4. Read the referenced spec from `_context/specs/` — focus on the current phase section
5. Read relevant `AGENTS.md` files for code conventions
6. Read relevant `PRD.md` sections
7. Scan affected files listed in the current phase's scope using `finder` / `Read`
8. If `loop_count > 0`: Read previous QA report and Progress Ledger — address the specific `instruction_or_question`

> **Bug Fix Fast Path note**: For tickets where `spec: none`, skip steps 4–6 (spec, AGENTS.md, PRD reading) and focus on the bug details in the Contract/Bug Details sections and the affected code.

### Step 3: Implement

#### Subagent Strategy (optional, for larger changes)
For changes spanning multiple services, consider spawning 2 parallel Tasks:
- **Task 1**: Service A impact analysis — scan affected files, check existing tests, identify dependencies
- **Task 2**: Service B impact analysis — scan affected components/modules, check contract alignment

Each returns a concise summary (max 300 words). The main agent then implements changes directly — subagents are for analysis only, never for writing code.

#### Service Conventions
Follow conventions from each service's `AGENTS.md` and `PRD.md`:
- Identify impacted service paths from handoff `context_keys` (for example: `apps/service-a`, `apps/service-b`)
- Use the stack conventions defined by that service (language, framework, architecture, test layout)
- Apply migration/process rules defined by that service for database/API/schema changes

**If writing React code:** Load the `vercel-react-best-practices` skill for performance optimization patterns (memoization, code splitting, data fetching, bundle size). Apply its guidelines to changed React code.

### Step 4: Self-Verify
Run checks before marking done. **Always use parallel subagents** — spawn one Task per service in a **single message**:

- **Task 1**: Run Service A checks (lint, format, typecheck, tests) and return a results summary (max 200 words)
- **Task 2**: Run Service B checks (lint, build, tests) and return a results summary (max 200 words)
- Add more Tasks if more services are impacted

**Do NOT run checks sequentially** — always use parallel subagents for speed.

**Service A checks (example):**
```bash
cd apps/service-a
ruff check src && ruff format --check src
mypy src
pytest tests/{relevant_test_files} -v
```

**Service B checks (example):**
```bash
cd apps/service-b
npm run lint
npm run build
npm run test
```

**React Check (if React files changed):**
Load the `vercel-react-best-practices` skill and run a focused audit against changed components. Fix any issues it flags before proceeding.

### Step 5: Elegance Gate
Before marking done, pause and self-review:
1. Ask: *"Would a staff engineer approve this diff?"*
2. For non-trivial changes: *"Is there a more elegant way to do this?"*
3. If the fix feels hacky: re-implement the clean solution now — do not defer.
4. Skip this step for simple, obvious fixes (single-line changes, config updates).

### Step 6: Update Handoff
1. Update handoff ticket: `status: pending` → `status: in-progress` → `status: done`
2. Add implementation notes to the handoff ticket:
   ```markdown
   ## Implementation Log
   - Phase: {current_phase} of {total_phases}
   - Files changed: {list}
   - Tests added: {list}
   - Migration: {yes/no, migration name}
   - Loop iteration: {loop_count} (if retrying after QA fail)
   - QA feedback addressed: {specific items from Progress Ledger, if applicable}
   - Notes: {anything QA should know}
   ```

### Step 7: Complete
Print summary:
```
✅ Implementation done: {title} (Phase {current_phase}/{total_phases})
📁 Files changed: {count}
🧪 Tests: {pass/fail summary}
⏭️ Next: Run /kd-qa to verify
```

## Rules
- Always read existing code before modifying
- Make smallest reasonable diffs
- Follow existing patterns — don't introduce new libraries without approval
- Write tests for new functionality
- Update PRD.md if adding new endpoints/features
- Never skip linting or type checking
- **Write to `_context/lessons.md`** when: (a) you discover a non-obvious root cause during debugging, (b) a QA loop reveals a recurring pattern, or (c) you find an undocumented codebase convention. Do not log routine implementation details.
