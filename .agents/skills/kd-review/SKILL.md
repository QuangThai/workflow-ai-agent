---
name: kd-review
description: "Structured, diff-aware code review gate for dev tickets. Produces blocking/non-blocking findings with severity, categories, file:line references, and fix suggestions. Triggers on: review, pre-landing review, audit diff."
---

# kd-review — Structured Review Gate

Run a focused review on the active ticket diff before dev marks work done.

## Workflow

### Step 1: Pick Up Target
1. Read `_context/lessons.md` for review-relevant patterns
2. Select ticket from `_handoff/queue/` where `to: dev` and `status: in-progress|done`
3. Read spec + implementation log + changed files

### Step 2: Run Diff-Aware Review
Review only changed scope and classify findings:
- Severity: `critical | high | medium | low`
- Category: `correctness | security | performance | reliability | ux | docs | test-gap`

Required output fields per finding:
- `severity`
- `category`
- `file:line`
- `issue`
- `why_it_matters`
- `suggested_fix`

### Step 3: Produce Review Report
Append to handoff ticket:

```markdown
## Review Report
- **Date**: {ISO date}
- **Agent**: review
- **Blocking findings**: {count} (`critical` + `high`)
- **Non-blocking findings**: {count} (`medium` + `low`)

| Severity | Category | Location | Issue | Suggested Fix |
|----------|----------|----------|-------|---------------|
| {severity} | {category} | {file:line} | {issue} | {fix} |
```

### Step 4: Update Ticket Gate Fields
Update frontmatter:
- `review_status: failed` if any blocking finding exists
- `review_status: passed` when no blocking finding remains
- `review_findings_summary: "blocking={N}, non_blocking={N}"`

## Rules
- Blocking findings (`critical`/`high`) must be fixed before `status: done`
- Always include file/line references for actionable output
- Do not block on cosmetic-only notes
