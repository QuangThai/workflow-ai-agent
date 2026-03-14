---
name: kd-health
description: "Validate workflow environment readiness: required files, queue/context folders, skill references, and optional tooling presence. Triggers on: health check, setup check, workflow readiness."
---

# kd-health — Workflow Health Check

Verify the orchestrator workspace is ready.

## Workflow

### Step 1: Structural Checks
Confirm required paths exist:
- `_context/`
- `_handoff/queue/` and `_handoff/archive/`
- `.agents/skills/`
- `.cursor/commands/`

### Step 2: Contract Checks
Validate:
- `_handoff/README.md` exists and includes ticket schema
- root `AGENTS.md` and `README.md` exist
- Core stage skills (`kd-brainstorm`..`kd-content`) are present

### Step 3: Utility Check
Validate utility skills/commands present:
- `kd-fix`, `kd-status`, `kd-review`, `kd-browser-qa`, `kd-ship`, `kd-health`

### Step 4: Browser QA Readiness Check
For each service repo under `apps/` (if present), check:
- `package.json` exists
- `@playwright/test` dependency present (devDependencies or dependencies)
- `playwright.config.*` exists

Report result as:
- `ready` (dependency + config found)
- `partial` (dependency found but no config, or vice versa)
- `missing` (no Playwright setup)

If status is `partial` or `missing`, include setup guidance:
- `.agents/skills/kd-browser-qa/references/playwright-setup-checklist.md`

### Step 5: Output
Produce a health report:

```markdown
## Workflow Health Report
- **Date**: {ISO date}
- **Overall**: healthy | degraded

| Check | Status | Notes |
|-------|--------|-------|
| Structure | ✅/❌ | {detail} |
| Contracts | ✅/❌ | {detail} |
| Core skills | ✅/❌ | {detail} |
| Utility skills | ✅/❌ | {detail} |
| Browser QA readiness | ✅/⚠️/❌ | {service-by-service status} |
```

## Rules
- Report concrete missing paths/files
- Do not mutate ticket/spec files during health checks
