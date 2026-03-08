---
name: kd-release
description: "Execute release process — deploy, verify, and update product state. Use when ready to deploy QA-passed changes. Triggers on: release, deploy, ship."
---

# kd-release — Release Agent

Execute the release process for Scopelytics AI.

## Workflow

### Step 1: Pick Up Release Ticket
1. Review `_context/lessons.md` for patterns relevant to deployment
2. List `_handoff/queue/` for tickets where `to: release` and `status: pending`
3. Read the release handoff ticket
4. Present release summary to user for final approval
5. **Fail-fast**: If no pending release tickets are found, STOP and report "No work ready for release." If the spec or QA report is missing, STOP and report.

### Step 2: Pre-Deploy Checklist
Verify all gates from PRDs:

**Backend (PRD §12):**
- [ ] `ruff check src` passes (no --fix: release must verify, not rewrite)
- [ ] `ruff format --check src` passes
- [ ] `mypy src` passes
- [ ] `pytest` passes with strict markers

**Frontend (PRD §11):**
- [ ] `npm run lint` passes
- [ ] `npm run build` passes
- [ ] `npm run check:api-contract` passes

### Step 3: Deploy
Reference the deploy script at `docs/deploy-scopelytics.sh`:
```bash
# The script handles:
# 1. Git pull on both repos
# 2. Backend permissions setup
# 3. Docker compose up (backend)
# 4. Docker build + run (frontend)
# 5. Health checks
bash docs/deploy-scopelytics.sh {branch}
```

Present the deploy command to user — **do not auto-execute deploy**.

### Step 4: Post-Deploy Verification
After user confirms deploy:
- [ ] Backend health: `curl http://127.0.0.1:8000/health`
- [ ] Frontend health: `curl http://127.0.0.1:3000`
- [ ] Specific smoke tests from release ticket
- [ ] Monitor for errors (check docker logs)

### Step 5: Update Product State
1. Update `_context/product-state.md`:
   - Find the line matching the spec ID in "Active Specs" and mark it with strikethrough + `released ✅`
   - Add a summary line to "Recent Decisions" with the date and outcome
2. Update spec status: `implemented` → `released`
3. Archive release handoff to `_handoff/archive/`

### Step 6: Create Content Handoff
Create content handoff in `_handoff/queue/`:

```markdown
---
id: HO-{next_id}  # (scan _handoff/queue/ and _handoff/archive/ per ID Allocation rules)
from: release
to: content
priority: {priority}
status: pending
created: {ISO timestamp}
spec: SPEC-XXX
total_phases: 1
current_phase: 1
loop_count: 0
output_mode: last_message
---

# Content: {Feature Title}

## Contract
- **task_description**: Generate content artifacts (changelog, blog, social, docs) for the shipped feature. Write for the target audience.
- **acceptance_criteria**: At least changelog entry and one blog/social draft produced. Content is accurate and references actual implementation.
- **context_keys**: _context/specs/SPEC-XXX-*.md, _context/product-state.md, _context/research/
- **output_mode**: last_message

## What Was Shipped
{User-facing summary}

## Key Features
- {Feature 1}
- {Feature 2}

## Screenshots / Demo Points
- {What to showcase}

## Technical Highlights
- {Interesting tech decisions for technical content}

## Target Audience
- {Who benefits from this feature}

## Content Suggestions
- [ ] Changelog entry
- [ ] Blog post / social post
- [ ] Documentation update
- [ ] Demo video script
```

### Step 7: Complete
```
🚀 Released: {title}
📋 Spec: SPEC-XXX (released)
📝 Content handoff created
⏭️ Next: Run /kd-content to create content
```

## Rules
- NEVER auto-deploy — always present command and wait for user approval
- Always verify health checks post-deploy
- Always create content handoff for shipped features
- Always include rollback commands in case of issues
- **Release must verify, not fix**: All pre-deploy checks must be non-mutating. If checks fail, route back to dev — do not auto-fix during release.
- **Fail-fast**: If required artifacts are missing, STOP and report rather than guessing.
