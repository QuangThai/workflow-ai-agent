---
name: kd-handoff-dev
description: "Prepare QA-passed work for release. Finalizes implementation notes, updates docs, and queues for release. Use after QA passes. Triggers on: handoff dev, prepare release, finalize."
---

# kd-handoff-dev — Dev → Release Handoff

Finalize QA-passed work and prepare it for release.

## Workflow

### Step 1: Verify QA Status
1. Read handoff ticket — confirm QA report shows `PASS` or `PASS-WITH-NOTES`
2. If not passed, redirect to `/kd-qa` or `/kd-dev`

### Step 2: Finalize Documentation
1. Verify `PRD.md` updates if new endpoints/features were added:
   - Backend: `scopelytics-ai-backend/PRD.md`
   - Frontend: `scopelytics-ai-frontend/PRD.md`
2. Verify `AGENTS.md` updates if architecture changed
3. Update spec status: `approved` → `implemented`

### Step 3: Create Release Handoff
Create release handoff in `_handoff/queue/`:

```markdown
---
id: HO-XXX
from: dev
to: release
priority: {priority}
status: pending
created: {ISO timestamp}
spec: SPEC-XXX
---

# Release: {Feature Title}

## Summary
{One-paragraph summary of what was built}

## Changes
### Backend
- {List of key changes with file paths}

### Frontend
- {List of key changes with file paths}

### Database
- Migration: {yes/no}
- Migration name: {if applicable}

## QA Status
- All automated checks: ✅
- Acceptance criteria: ✅
- QA report: {link to handoff ticket}

## Deploy Notes
- Environment variables: {any new env vars}
- Migration steps: {if applicable}
- Breaking changes: {if any}
- Rollback plan: {how to rollback if needed}

## Post-Deploy Verification
- [ ] Health check passes
- [ ] Smoke test: {specific test}
- [ ] Monitor: {what to watch}
```

### Step 4: Archive Original Handoff
Move the original dev handoff ticket from `_handoff/queue/` to `_handoff/archive/`

### Step 5: Complete
```
✅ Release handoff ready: _handoff/queue/{filename}
📋 Spec: SPEC-XXX (implemented)
🚀 Next: Run /kd-release to deploy
```

## Rules
- Never skip documentation updates
- Always include rollback plan
- Always archive completed handoffs
