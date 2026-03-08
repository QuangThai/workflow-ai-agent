---
name: kd-qa
description: "Run QA verification on completed dev work. Checks tests, linting, types, contracts, and acceptance criteria. Use after dev completes implementation. Triggers on: qa, verify, test, check quality."
---

# kd-qa — Quality Assurance Agent

Verify completed dev work against specs, run tests, and ensure quality standards.

## Workflow

### Step 1: Pick Up Completed Work
1. List `_handoff/queue/` for tickets where `status: done` and `to: dev` (completed by dev)
2. Read the handoff ticket and referenced spec
3. Read the implementation log from dev

### Step 2: Automated Checks

#### Backend Verification
```bash
cd scopelytics-ai-backend
# Lint & format
ruff check src
ruff format --check src
# Type check
mypy src
# Full test suite
pytest -v
# Security regression (if security-related changes)
pytest tests/test_guardrails.py tests/test_guardrails_security.py tests/test_prompt_security.py -v
```

#### Frontend Verification
```bash
cd scopelytics-ai-frontend
# Lint
npm run lint
# Build (includes typecheck)
npm run build
# API contract
npm run check:api-contract
```

### Step 3: Manual Review Checklist
For each changed file, verify:
- [ ] Code follows conventions (check `AGENTS.md`)
- [ ] No hardcoded secrets or credentials
- [ ] Error handling is appropriate
- [ ] Types are correct and complete
- [ ] Tests cover the new functionality
- [ ] No unnecessary changes outside scope

### Step 4: Acceptance Criteria Verification
Go through each acceptance criterion from the spec:
- [ ] Criterion 1: {PASS/FAIL — evidence}
- [ ] Criterion 2: {PASS/FAIL — evidence}

### Step 5: Generate QA Report
Create QA report in the handoff ticket:

```markdown
## QA Report
- **Date**: {ISO date}
- **Agent**: qa
- **Verdict**: PASS | FAIL | PASS-WITH-NOTES

### Automated Checks
| Check | Result |
|-------|--------|
| Backend lint | ✅/❌ |
| Backend types | ✅/❌ |
| Backend tests | ✅/❌ ({pass}/{total}) |
| Frontend lint | ✅/❌ |
| Frontend build | ✅/❌ |
| API contract | ✅/❌ |

### Acceptance Criteria
| Criterion | Result | Evidence |
|-----------|--------|----------|
| {criterion} | ✅/❌ | {detail} |

### Issues Found
{List any issues, or "None"}

### Notes for Release
{Anything the release agent should know}
```

### Step 6: Route Result
**If PASS:**
- Print: `✅ QA passed. Run /kd-handoff-dev to prepare for release.`

**If FAIL:**
- Create feedback handoff back to dev in `_handoff/queue/`:
  - `from: qa`, `to: dev`, with specific issues to fix
- Print: `❌ QA failed. Issues queued back to dev. Run /kd-dev to fix.`

## Rules
- Always run ALL automated checks, not just the ones related to the change
- Be thorough but fair — only flag real issues
- Provide evidence for every fail
- Update `_context/metrics/` with quality data
