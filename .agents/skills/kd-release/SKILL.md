---
name: kd-release
description: "Use after /kd-handoff-dev when QA-passed work is ready to go live. Confirm whether the change is already live to its intended audience (production, package registry, app store, internal rollout), verify release evidence, update product state, and create the content handoff. Triggers on: release, deploy, ship, 'is this deployed?', 'is this live?', 'ready to go live', 'post-deploy verification'. Does not execute deploy commands — deployment happens outside this workflow."
---

# kd-release — Release Agent

Verify that shipped work is live (or ready to go live), confirm release status with the user, update product state, and hand off to content generation.

This agent does NOT deploy code. Deployment happens outside this workflow — via CI/CD, platform auto-deploy, `npm publish`, app store submission, manual scripts, or whatever your team uses. This agent's job is to **verify, record, and transition**.

## Core Principle

**Ask, don't assume.** Every team releases differently. A SaaS team merges to main and CI/CD handles the rest. A library author runs `npm publish`. A mobile team submits to TestFlight then App Store. This agent asks the user what's true, then acts on the answers.

---

## Workflow

### Step 1: Pick Up Release Ticket

Pick up a release ticket in this priority order:

1. **Resume first**: Look for tickets with `to: release`, `status: in-progress`, and `deploy_status: awaiting-user-deploy` — the user previously ran this step and went off to deploy. Resume from Step 2 with the assumption they're back because the deploy is done.
2. **New tickets**: Look for tickets with `to: release` and `status: pending`.
3. **Multiple candidates**: If more than one ticket matches, list them and ask the user which one to work on.

Then:
1. Review `_context/lessons.md` for patterns relevant to releases
2. Read the release handoff ticket
3. Update ticket `status: pending` → `status: in-progress` (if not already in-progress)
4. **Fail-fast**: If no matching release tickets found → STOP and report "No work ready for release." If the spec or QA report is missing → set `status: blocked` and STOP.

### Step 2: Release Status Check

Ask the user these questions to understand the current state. The goal is to determine whether the change is **live to its intended audience** — not just "deployed somewhere."

```
Before we proceed, I need to understand the release situation:

1. **Is this change live to its intended audience?**
   - (a) Yes — live in production (CI/CD auto-deployed on merge, or manually deployed)
   - (b) Yes — published to registry / app store / distribution channel
   - (c) Partially — deployed to staging / preview / beta / TestFlight only
   - (d) No — not deployed or published yet
   - (e) No deployment needed — internal tool / local-only / documentation change

2. **Where is it live?** (or where will it be)
   - Environment, URL, registry, store, etc.
   - Or "N/A" if no deployment needed

3. **Any issues observed since it went live?** (if already live)
   - Or "N/A" if not yet live
```

**Route based on answers:**

| Answer | Action |
|--------|--------|
| **(a) Live in production** | → Step 4 (Post-Release Verification) |
| **(b) Published to channel** | → Step 4 (Post-Release Verification) |
| **(c) Staging/beta only** | → Step 3c (Not Yet Released) — staging is not released |
| **(d) Not deployed yet** | → Step 3 (Pre-Release Readiness) |
| **(e) No deploy needed** | → Step 5 (Update Product State) — record reason |

**If Q3 reports issues**: Feed directly into Step 4 verification — these are pre-known failures to investigate.

**Key distinction**: "deployed to staging" is NOT "released." This agent only marks a spec as `released` when the change is **live to the intended audience** (production users, registry consumers, app store users, internal stakeholders — whoever the feature is for).

### Step 3: Pre-Release Readiness (only if not yet deployed/published)

Present a readiness checklist based on the release ticket. Do NOT run deploy commands — help the user verify they're ready to deploy using their own process.

**Readiness checklist (adapt to what's in the release ticket — don't invent generic checks):**

```
Pre-release checklist for {Feature Title}:

Environment & Config:
- [ ] Required env vars set in target environment
      {list specific vars from release ticket}
- [ ] Feature flags configured (if applicable)
- [ ] Database migrations ready (if applicable)

Quality Gates (from QA report):
- [ ] QA verdict: PASS (Full Gate)
- [ ] All acceptance criteria verified
- [ ] No known blocking issues

Deploy Order (for multi-service changes):
- [ ] {service-1} first, then {service-2}
      (from release ticket deploy order)

Rollback Plan:
- [ ] Rollback steps documented: {summarize from ticket}
```

**Then tell the user:**
```
When you're ready, deploy/publish using your usual process.
Then run /kd-release again — I'll pick up where we left off and verify the release.
```

Update the ticket:
- Keep `status: in-progress`
- Add `deploy_status: awaiting-user-deploy`
- Record target environment/channel from Q2

**STOP** — the user will re-run `/kd-release` after deploying.

### Step 3c: Not Yet Released (staging/beta/preview only)

The change is deployed to a non-final channel. This is useful for verification, but the spec should not be marked `released` yet.

```
Your change is on {staging/beta/preview}, but not yet live to the intended audience.

Options:
1. **Verify staging and wait** — I'll record the staging verification, then you
   promote to production/release later and re-run /kd-release.
2. **This IS the intended channel** — if staging/beta/preview is the final
   destination (e.g., internal tool, beta program), let me know and I'll
   treat it as released.
```

If the user chooses option 1:
- Run Step 4 verification checks against the staging environment
- Record results in the ticket with `deploy_status: verified-staging`
- Keep `status: in-progress`
- **STOP** — user will re-run after promoting to production

If the user chooses option 2:
- Proceed to Step 4 → Step 5 → Step 6 as normal (this channel IS the release target)

### Step 4: Post-Release Verification

Confirm the release is healthy. Adapt checks based on what's in the release ticket and the type of release — don't invent checks that weren't specified.

**For services/SaaS:**
```
1. Health check: Is the service responding?
   - {specific endpoint or URL from ticket, if available}
2. Smoke test: Can you verify the core feature works?
   - {specific test from acceptance criteria}
3. Error monitoring: Any new errors in logs/dashboard?
4. User-reported issues: Any reports since deploy?
```

**For libraries/packages:**
```
1. Package available: Can you install the published version?
   - {registry URL or install command}
2. Basic usage: Does the package work as expected?
3. Version correct: Is the published version number right?
```

**For mobile/desktop apps:**
```
1. Build available: Is the build available in the target track?
2. Install test: Can you install and launch it?
3. Crash-free: Any crashes reported?
```

If Q3 from Step 2 reported pre-known issues, address those first.

**Record the evidence in the ticket:**

```markdown
## Release Verification
- **Release method**: {CI/CD | manual deploy | npm publish | app store | platform | etc.}
- **Channel**: {production | npm latest | App Store | internal | etc.}
- **Environment**: {URL, registry, store link}
- **Released at**: {ISO timestamp, from user}
- **Verified at**: {now}

| Check | Result | Notes |
|-------|--------|-------|
| {check 1} | ✅/❌ | {details} |
| {check 2} | ✅/❌ | {details} |
| {check 3} | ✅/❌ | {details} |
```

**If verification fails:**
1. Ask the user: "Do you want to rollback, or investigate first?"
2. If rollback — remind them of the rollback steps from the release ticket
3. Update the current release ticket: `deploy_status: failed-verification`
4. Create a feedback ticket in `_handoff/queue/`:

```markdown
---
id: HO-{next_id}
from: release
to: dev
priority: P0
status: pending
created: {ISO timestamp}
spec: SPEC-XXX
total_phases: 1
current_phase: 1
loop_count: 0
output_mode: full_history
---

# Rollback: {Feature Title} — Post-Release Issue

## Contract
- **task_description**: Post-release verification failed for {Feature Title}. Investigate root cause and fix.
- **acceptance_criteria**: All post-release checks pass after fix. No regressions.
- **context_keys**: {spec, release ticket, QA report}
- **output_mode**: full_history

## Issue Details
- {Which checks failed and how}
- {User-reported symptoms from Q3}
- {Related release ticket}: HO-{release_ticket_id}

## Current State
- Rollback executed: {yes/no}
- Environment state: {what state the system is in now}
```

Print: `🚨 Post-release verification failed. Release ticket marked failed-verification. Feedback ticket HO-{id} created for dev.`
**STOP** — do not proceed to content handoff.

### Step 5: Update Product State
1. Update `_context/product-state.md`:
   - Remove or strikethrough the spec line in **Active Specs**
   - Add a deterministic release line to **Recent Decisions**:
     ```
     - **{YYYY-MM-DD}** [SPEC-XXX] {Feature Title} released to {channel} via {method}. (content pending)
     ```
2. Update spec status → `released`
3. Update ticket `status: in-progress` → `status: done`
4. Archive release handoff to `_handoff/archive/`

### Step 6: Create Content Handoff

Determine announcement scope based on what was released:

| Release type | `announcement_scope` |
|-------------|----------------------|
| Production / public registry / app store | `public` |
| Customer-specific / enterprise deploy | `customer` |
| Internal tool / staging-only / beta | `internal` |
| Documentation / config-only change | `none` |

Create content handoff in `_handoff/queue/`:

```markdown
---
id: HO-{next_id}
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
announcement_scope: {public | customer | internal | none}
---

# Content: {Feature Title}

## Contract
- **task_description**: Generate publication-ready content artifacts for the shipped feature. Build a release brief first, then generate changelog, release notes, social posts, and any justified long-form content.
- **acceptance_criteria**: (1) Release brief created with concrete evidence. (2) Required artifacts generated based on announcement_scope. (3) Content quality review passes. (4) All content is evidence-backed.
- **context_keys**: _context/specs/SPEC-XXX-*.md, _context/product-state.md, _handoff/archive/
- **output_mode**: last_message

## What Was Shipped
{User-facing summary — concrete capabilities, not plans}

## Key Features
- {Feature 1 — specific, not abstract}
- {Feature 2}
- {Feature 3}

## User Problem / Prior State
- **Before**: {how things worked before — specific friction, workarounds, risks}
- **Pain points**: {what users experienced or worked around}

## What Changed for Users
- **After**: {new workflow, fewer steps, better outcome}
- **Impact**: {time saved, steps removed, risks eliminated}

## Implementation Evidence
- **Files changed**: {key files from dev implementation log}
- **Tests**: {from QA report — e.g., "8/8 passed across 3 suites"}
- **QA verdict**: {PASS/PASS-WITH-NOTES — copy from QA report}
- **Release method**: {CI/CD | manual | npm publish | etc.}
- **Release channel**: {production | npm latest | App Store | etc.}

## Verification Evidence
- **Release ticket**: `_handoff/archive/HO-{id}-*.md`
- **QA report**: {exact location — in archived handoff or separate file}
- **Release verification**: {summary from Step 4 — checks passed, timestamps}

## API Changes (if any)
- {New/changed endpoints, request/response — or "None"}
- {Auth/permission changes — or "None"}

## Breaking Changes / Migration
- {Breaking changes — or "None"}
- {Migration steps — or "None"}
- {Deprecations — or "None"}

## Known Limitations
- {Honest constraints — what doesn't work yet}
- {Follow-up work planned}

## Release Scope
- **Announcement scope**: {public | customer | internal | none}
- **Channel**: {where it's live}
- **Audience**: who can access it now

## Target Audience
- **Primary**: {who directly benefits}
- **Secondary**: {who indirectly benefits}
```

**Announcement scope determines which artifacts `kd-content` should generate:**

| Scope | Required artifacts | Optional artifacts |
|-------|-------------------|-------------------|
| `public` | changelog, release notes, social (X, LinkedIn, Discord) | blog, engineering post, API docs, email |
| `customer` | changelog, release notes, email | blog (if meaningful story) |
| `internal` | changelog | release notes (simplified) |
| `none` | changelog only | — |

### Step 7: Complete
```
🚀 Released: {title}
📋 Spec: SPEC-XXX → released
📝 Content handoff created: HO-{id} (scope: {announcement_scope})
⏭️ Next: Run /kd-content to generate release content
```

---

## Rules
- **NEVER deploy code** — this agent verifies and records, it does not execute deployments
- **Always ask** — do not assume deployment status, environment, or method
- **"Deployed" ≠ "released"** — only mark `released` when live to the intended audience, not just staging/preview
- **Adapt checklists** — use what's in the release ticket, because invented checks create false confidence
- **Fail-fast** — if required artifacts are missing, STOP and report
- **Two-pass for not-yet-deployed** — present readiness checklist, STOP, user re-runs after deploying. Resume via `deploy_status: awaiting-user-deploy`
- **Record everything** — release method, channel, timestamps, verification results go into the ticket
- **Clean up on failure** — if verification fails, mark release ticket `failed-verification` and link the dev feedback ticket. Never leave orphaned in-progress tickets
