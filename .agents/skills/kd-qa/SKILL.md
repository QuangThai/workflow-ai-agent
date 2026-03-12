---
name: kd-qa
description: "Run QA verification on completed dev work. Checks tests, linting, types, contracts, and acceptance criteria. Use after dev completes implementation. Triggers on: qa, verify, test, check quality."
---

# kd-qa — Quality Assurance Agent

Verify completed dev work against specs, run tests, and ensure quality standards.

## Workflow

### Step 1: Pick Up Completed Work
1. Review `_context/lessons.md` for patterns relevant to QA tasks
2. List `_handoff/queue/` for tickets where `to: dev` and `status: done` — these are dev-completed tickets ready for QA review
3. Read the handoff ticket, referenced spec, and implementation log from dev
4. Identify `current_phase` and `total_phases` from frontmatter
5. **Fail-fast**: If no completed tickets are found, STOP and report "No completed work ready for QA." If the referenced spec is missing, STOP and report the dependency.

### Step 2: Automated Checks

Use a two-tier QA gate to improve delivery speed without dropping safety:
- **Fast Gate (default per dev ticket)**: lint + typecheck + impacted tests + changed-scope contract checks
- **Full Gate (required before release)**: full regression suites for all impacted services

#### Subagent Strategy (required)
**Always** spawn parallel Task calls (one per service) in a **single message** to run all checks concurrently:
- **Task 1**: Run all checks for service A (lint, typecheck, tests) and return a summary table of results
- **Task 2**: Run all checks for service B (lint, build, tests) and return a summary table of results
- Add more Tasks if more services are impacted

Each Task returns ONLY the results table (max 200 words). The main agent then proceeds to manual review and acceptance criteria.

**Do NOT run checks sequentially** — always use parallel subagents for speed.

#### Service A Verification (example)
```bash
cd apps/service-a
# Lint & format
ruff check src
ruff format --check src
# Type check
mypy src
# Fast Gate: impacted tests
pytest tests/{changed_or_related_tests} -v
# Full Gate (required before release): full suite
# pytest -v
```

#### Service B Verification (example)
```bash
cd apps/service-b
# Lint
npm run lint
# Build (includes typecheck)
npm run build
# Fast Gate: scoped tests/checks
npm run test -- --changed
# Full Gate (required before release): full suite
# npm run test
```

#### React Audit (if React files changed)
Load the `vercel-react-best-practices` skill and run a focused audit against changed components. Verify performance patterns, hook usage, and component structure. Add findings to the QA Report.

### Step 3: Manual Review Checklist
For each changed file, verify:
- [ ] Code follows conventions (check `AGENTS.md`)
- [ ] No hardcoded secrets or credentials
- [ ] Error handling is appropriate
- [ ] Types are correct and complete
- [ ] Tests cover the new functionality
- [ ] No unnecessary changes outside scope

### Step 4: Acceptance Criteria Verification
Verify acceptance criteria based on the current phase:

**Current phase criteria** (must all pass):
- [ ] Each acceptance criterion for the current phase: {PASS/FAIL — evidence}

**Regression check** (if current_phase > 1):
- [ ] Prior phase functionality still works: {PASS/FAIL — evidence}

**Full spec criteria** (only on final phase, when current_phase == total_phases):
- [ ] Each overall spec acceptance criterion: {PASS/FAIL — evidence}

### Step 5: Generate QA Report
Create QA report in the handoff ticket:

```markdown
## QA Report
- **Date**: {ISO date}
- **Agent**: qa
- **Verdict**: PASS | FAIL | PASS-WITH-NOTES
- **Phase**: {current_phase} of {total_phases}

### Automated Checks
| Check | Result |
|-------|--------|
| Gate Mode (Fast/Full) | ✅/❌ |
| Service A lint | ✅/❌ |
| Service A types | ✅/❌ |
| Service A tests | ✅/❌ ({pass}/{total}) |
| Service B lint | ✅/❌ |
| Service B build | ✅/❌ |
| Service B tests/contracts | ✅/❌ |

### Acceptance Criteria
| Criterion | Result | Evidence |
|-----------|--------|----------|
| {criterion} | ✅/❌ | {detail} |

### Issues Found
{List any issues, or "None"}

### Notes for Release
{Anything the release agent should know}
```

**Verdict definitions:**
- `PASS`: All checks green, all acceptance criteria met
- `PASS-WITH-NOTES`: All criteria met, but cosmetic observations exist (naming suggestions, minor style notes). Notes must NOT require code changes — if they do, verdict must be `FAIL`
- `FAIL`: Any check failed or acceptance criterion not met

### Step 6: Progress Ledger Check
Before routing the result, evaluate progress to detect loops and stalls.

Fill out the progress ledger:

```markdown
## Progress Ledger
- **is_request_satisfied**:
  - reason: "{Evidence-based reasoning}"
  - answer: true/false
- **is_in_loop**:
  - reason: "{Compare current issues with previous QA feedback — are the same issues recurring?}"
  - answer: true/false
- **is_progress_being_made**:
  - reason: "{Did the dev agent address the specific issues from last round?}"
  - answer: true/false
- **loop_count**: {N — number of QA→dev→QA cycles for this spec/phase}
- **instruction_or_question**:
  - reason: "{What specifically should the dev agent do differently?}"
  - answer: "{Concrete, actionable instruction — not 'fix it', but 'change X in file Y to handle case Z'}"
```

**Loop detection rules:**
- `loop_count >= 3`: **STOP.** Escalate to user with full context (all QA reports + dev logs). The problem likely requires human judgment or a spec revision.
- `is_in_loop == true` AND `loop_count >= 2`: **Re-plan.** The fix approach is wrong. Update the spec or handoff with a new technical approach before retrying.
- `is_progress_being_made == false`: Flag the specific blocker. If it's an architectural issue, escalate. If it's a misunderstanding, provide explicit code-level guidance.

### Step 7: Route Result
**If PASS:**
- Print: `✅ QA passed. Run /kd-handoff-dev to prepare for release.`

**If FAIL (and loop_count < 3):**
- **Reuse the existing ticket** — do NOT create a new one:
  1. Append the QA Report and Progress Ledger to the existing ticket
  2. Reset `status: done` → `status: pending` (so dev picks it up again)
  3. Increment `loop_count` in the frontmatter
  4. The `instruction_or_question` from the Progress Ledger tells dev exactly what to fix
- Print: `❌ QA failed (loop {loop_count}). Ticket returned to dev. Run /kd-dev to fix.`

**If FAIL (and loop_count >= 3):**
- Print: `🚨 QA loop detected ({loop_count} cycles). Escalating to user.`
- Present full QA history and recommend: re-spec, pair debugging, or scope reduction

## Rules
- Always run all checks required by the selected gate mode
- Full Gate is mandatory before `/kd-handoff-dev` and `/kd-release`
- Be thorough but fair — only flag real issues
- Provide evidence for every fail
- Update `_context/metrics/` with quality data
- **Write to `_context/lessons.md`** when: (a) the same failure pattern recurs across multiple specs, (b) a QA check reveals a gap in the pipeline, or (c) `loop_count >= 2` exposes a systemic issue. Do not log routine pass/fail results.
