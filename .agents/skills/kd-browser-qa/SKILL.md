---
name: kd-browser-qa
description: "Capture browser QA evidence for UI/web changes (screenshots, logs, flow checks) and attach artifact paths to handoff tickets. Triggers on: browser qa, ui qa, screenshot evidence."
---

# kd-browser-qa — Browser Evidence QA (Playwright-First)

Collect reproducible browser evidence for UI/web changes.

## Workflow

### Step 1: Scope
1. Read target ticket and identify changed routes/components
2. Define scenarios:
   - Happy path
   - Primary error state
   - Empty/edge state (if applicable)

### Step 2: Select Execution Mode
Prefer Playwright automation first.

**Mode A — Playwright (default):**
Use when target service has:
- `@playwright/test` dependency, or
- `playwright.config.{ts,js,mjs,cjs}`

Execution guidance:
1. Run browser QA tests:
   - `npx playwright test --reporter=line`
   - For targeted flows, run scoped specs/grep from the handoff scope.
2. Ensure trace/screenshot/video artifacts are generated (default Playwright outputs are acceptable).
3. Collect artifacts from `test-results/` and/or `playwright-report/`.

**Mode B — Manual browser fallback:**
Use only if Playwright is not configured in the target service.
If readiness is `missing`/`partial`, first read and apply `references/playwright-setup-checklist.md`.
Capture:
- screenshot(s) per scenario
- repro steps
- console/network errors

### Step 3: Capture Evidence
For each scenario capture:
- Screenshot(s)
- Short step trace
- Console/network error notes

Save artifacts under:
- `_context/metrics/qa/{HO-ID}/browser/`

### Step 4: Report
Append to ticket:

```markdown
## Browser QA Report
- **Agent**: browser-qa
- **Date**: {ISO date}

| Scenario | Result | Evidence Path | Notes |
|----------|--------|---------------|-------|
| {name} | ✅/❌ | {_context/metrics/qa/{HO-ID}/browser/...} | {summary} |
```

Update `evidence_paths` in frontmatter with all generated paths.
Include mode used: `playwright` or `manual-fallback`.

## Rules
- Do not mark UI evidence complete without at least one happy-path screenshot
- Prefer Playwright mode whenever available; manual mode is fallback only
- When Playwright readiness is `missing` or `partial`, provide the setup checklist path in the QA note:
  - `.agents/skills/kd-browser-qa/references/playwright-setup-checklist.md`
- If scenario fails, report exact repro steps and error text
