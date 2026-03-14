# Playwright Setup Checklist (Standard for `apps/*`)

Use this checklist when Browser QA readiness is `partial` or `missing`.

## 1) Install

Run in target service repo:

```bash
npm install -D @playwright/test
npx playwright install
```

If service uses pnpm/yarn, use equivalent package manager commands.

## 2) Initialize Config

Create `playwright.config.ts` (or `.js`) at service root with:
- `testDir` pointing to E2E folder
- `use.trace = "on-first-retry"`
- `use.screenshot = "only-on-failure"`
- `use.video = "retain-on-failure"`
- baseURL from service env (local dev URL)

## 3) Add Standard Test Location

Create:
- `tests/e2e/`
- at least one smoke spec, e.g. `tests/e2e/smoke.spec.ts`

Minimum scenarios:
- happy path
- primary error state

## 4) Add Scripts to `package.json`

```json
{
  "scripts": {
    "test:e2e": "playwright test",
    "test:e2e:headed": "playwright test --headed",
    "test:e2e:report": "playwright show-report"
  }
}
```

## 5) Artifact Mapping to Workflow

After running E2E, copy/record artifacts into:
- `_context/metrics/qa/{HO-ID}/browser/`

Expected artifacts:
- screenshots
- traces (`.zip`)
- short scenario result summary

Then update ticket:
- `evidence_paths: [...]`
- `Browser QA Report` section with pass/fail per scenario

## 6) Validation Gate

Setup is considered complete when:
- `@playwright/test` is present
- `playwright.config.*` exists
- at least one E2E smoke spec exists
- `npm run test:e2e` (or equivalent) executes successfully
