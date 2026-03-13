# Content Playbooks — Artifact Templates & Rules

This file contains the detailed templates and rules for each content artifact type.
Read only the sections relevant to the artifacts you're generating.

## Table of Contents
- [A. Changelog](#a-changelog)
- [B. Release Notes](#b-release-notes)
- [C. Blog Post](#c-blog-post)
- [D. Social Package](#d-social-package)
- [E. API Documentation Draft](#e-api-documentation-draft)
- [F. Engineering Post](#f-engineering-post)
- [G. Email Announcement](#g-email-announcement)

---

## A. Changelog — `changelog.md`
**Audience**: developers, operators, integrators, maintainers.
**Purpose**: factual, exhaustive record of what changed.

```markdown
[agent: content]

# Changelog

## [{version}] — {YYYY-MM-DD}

### Summary
{2-4 sentences: what shipped, why it matters, who is affected}

### Highlights
- {Most important change — concrete, specific}
- {Second important change}
- {Third important change}

### Added
- {New capability with enough detail to understand without reading docs}
- {New endpoint: `POST /api/v1/resource` — purpose and key params}
- {New config option: `FEATURE_FLAG` — what it controls}

### Changed
- {Behavior change — what was the old behavior, what is the new behavior}
- {Performance improvement — with numbers if measured}

### Fixed
- {Bug fix — what symptom users saw, what was the root cause}
- {Edge case handled — specific scenario}

### Breaking Changes
{Omit section if none}
- **{change}**: {old behavior} → {new behavior}
  - Migration: {exact steps to update}
  - Affected: {which users/integrations are impacted}

### Deprecations
{Omit section if none}
- `{deprecated_item}` — will be removed in {version/date}. Use `{replacement}` instead.

### Migration / Upgrade Guide
{Omit section if no action required}
1. {Step — be specific about files, configs, commands}
2. {Step}
3. **Verify**: {how to confirm upgrade succeeded}

### Security
{Omit section if no security-relevant changes}
- {Security improvement or fix — without exposing vulnerability details}

### Dependencies
{Omit section if no dependency changes}
- Added: `{package}@{version}` — {why}
- Updated: `{package}` {old} → {new} — {why}
- Removed: `{package}` — {why}

### Known Limitations
{Omit section if none}
- {Honest constraint — what doesn't work yet or has caveats}

### References
- Spec: `_context/specs/SPEC-XXX-*.md`
- Docs: {link placeholders}
- Release brief: `_context/content/YYYY-MM-DD-{slug}/release-brief.md`
```

**Rules**:
- Every entry must describe user-visible or operator-visible impact
- Include migration notes for any behavior change
- Prefer exact facts over marketing language
- Do NOT leave empty sections — omit them entirely

---

## B. Release Notes — `release-notes.md`
**Audience**: end users, product managers, non-technical stakeholders.
**Purpose**: plain-language summary of what changed and why users should care.

```markdown
[agent: content]

# Release Notes — {Version or Feature Name}

## What's New
{2-3 sentence overview in plain language — focus on outcome, not implementation}

## What You'll Notice
- ✨ {User-visible benefit 1 — describe the experience, not the code}
- 🔧 {User-visible benefit 2}
- ⚡ {User-visible benefit 3}

## Before → After
| | Before | After |
|---|---|---|
| {Workflow/task} | {old experience} | {new experience} |
| {Another task} | {old experience} | {new experience} |

## How to Get Started
1. {First step — simple, actionable}
2. {Second step}
3. {How to verify it's working}

## If You're Upgrading
{Omit section if no user action required}
- {Simple upgrade instruction}
- {Breaking change in plain language, if any}
- {Who is affected and who is not}

## Availability
- {Plan/tier/role/region/platform if relevant, or "Available to all users"}

## Frequently Asked Questions
{Include 2-3 anticipated questions}
**Q: {question}?**
A: {concise answer}

## Learn More
- Documentation: {link placeholder}
- Support: {link placeholder}
- Feedback: {link placeholder}
```

**Rules**:
- Focus on user outcome, not architecture
- Avoid internal jargon — explain technical terms if needed
- Keep it skimmable with clear headers and bullets
- Before→After table is required when workflow changed

---

## C. Blog Post — `blog-post.md`
**Audience**: developers, technical leaders, community.
**Purpose**: publication-quality long-form piece that teaches something.

Only generate if the release has a meaningful story. No blog post is better than a generic one.

```markdown
[agent: content]

# {Specific outcome-driven title — not "Introducing Feature X"}

> {1-sentence dek/subtitle — the promise of the article}

## The problem we kept running into
{Tell a concrete scenario — a real user/developer situation, not abstract pain.
Describe the friction, the workaround, the cost of the status quo.
Make the reader nod and think "yes, I've been there."}

## What we built
{Describe the shipped capability in clear terms.
Focus on what it enables, not what it is internally.}

## Before vs. After

### Before
{Old workflow step by step — highlight friction points}
- Step 1: {tedious/error-prone step}
- Step 2: {manual workaround}
- Step 3: {fragile outcome}

### After
{New workflow step by step — highlight improvements}
- Step 1: {simplified step}
- Step 2: {automated/eliminated step}
- Result: {reliable outcome}

## A concrete example

{Show a real usage example — code snippet, API call, CLI command, or UI walkthrough.
This section must contain at least one code block or step-by-step demonstration.}

```{language}
// Example code showing the feature in action
{realistic, working example}
```

{Explain what the example demonstrates and why it matters.}

## How it works (the interesting parts)
{Enough implementation detail to build trust and educate.
Architecture diagram suggestion, key design decisions.
NOT a full code walkthrough — pick 1-2 interesting details.}

> 💡 **Visual opportunity**: {Describe a diagram or screenshot that would enhance this section}

## What to know when upgrading
{Migration notes, compatibility, limits — honest and specific.
Omit section if no action required.}

## Who should use this
{Ideal users, teams, use cases — be specific.}

## What's next
{Only real planned follow-ups confirmed in the spec. Not a generic roadmap.
Omit section if nothing is confirmed.}

---

*{Feature slug} is available now. [Get started →]({link placeholder})*
```

**Rules**:
- Must have a narrative arc — not "Problem / Solution / How It Works" by default
- Must include at least one concrete example (code, API, or workflow)
- If you cannot show a real example, do NOT generate the blog post
- Suggest visuals (diagrams, screenshots, before/after) — mark with 💡
- Do not repeat the changelog — this tells a story, not a list

---

## D. Social Package — Channel-Native Posts

Each social post must feel native to its platform. Do NOT copy-paste the same content across channels.

### `social-x-thread.md` — X / Twitter Thread
**Tone**: punchy, insight-driven, scroll-stopping.

```markdown
[agent: content]

# X Thread — {Feature Title}

## Thread (5-8 posts)

### 1/N — Hook
{Start with an outcome, surprising pain point, or bold claim.
Not "We just shipped X." Instead: "We spent 3 weeks debugging auth redirect loops. Here's what we learned and built."}

### 2/N — The pain
{Describe the problem in 1-2 sentences. Be specific.}

### 3/N — What shipped
{What the feature does — concrete, not abstract.}

### 4/N — Why it matters
{User impact, time saved, risks eliminated.}

### 5/N — Show, don't tell
{Concrete example — code snippet, screenshot description, or workflow.}
```{language}
{short code example if applicable}
```

### 6/N — The interesting technical detail
{One architectural decision or trade-off that developers would find interesting.}

### 7/N — Migration / compatibility note
{Omit if not relevant. Include if users need to act.}

### 8/N — CTA
{Try it → {link placeholder}
Docs → {link placeholder}
Feedback welcome 🙏}
```

### `social-linkedin.md` — LinkedIn Post
**Tone**: professional, thoughtful, credible — not hypey.

```markdown
[agent: content]

# LinkedIn Post — {Feature Title}

{Hook — start from an observed problem or industry pattern, not a product announcement.}

{What we shipped and why it matters to teams — 2-3 sentences.}

Key highlights:
• {Business/user impact 1}
• {Business/user impact 2}
• {Technical insight or lesson learned}

{Short reflection — what we learned building this, what surprised us.}

{CTA — link placeholder, invite discussion.}

---
Word count target: 180-350 words
```

### `social-discord-slack.md` — Discord / Slack Announcement
**Tone**: concise, high-signal, community-friendly.

```markdown
[agent: content]

# Discord / Slack Announcement — {Feature Title}

🚀 **{Feature Title}** is live!

{1-sentence description of what shipped.}

**What's new:**
- {Key capability 1}
- {Key capability 2}
- {Key capability 3}

**Upgrade note:** {1 line — or "No action needed"}

📖 Docs: {link placeholder}
💬 Questions? Drop them in this thread!

---
Word count target: 60-140 words
```

**Rules for all social**:
- Do NOT reuse the same opening line across channels
- Do NOT paste blog paragraphs into social posts
- Each channel must feel native to its platform
- Include at least one concrete detail (not just "we improved performance")

---

## E. API Documentation Draft — `api-doc-draft.md`
**Generate only if API surface changed** (new endpoints, changed request/response, auth changes).

```markdown
[agent: content]

# API Documentation — {Feature/API Name}

## Overview
{What the API does, when to use it, who it's for}

## Authentication
{Auth model — headers, tokens, scopes, roles}

## Endpoints

### `{METHOD} {/path}`
**Purpose**: {what this endpoint does}

**Request:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| {param} | {type} | {yes/no} | {description} |

**Response** (`{status_code}`):
```json
{realistic response body}
```

**Errors:**
| Code | Meaning | Resolution |
|------|---------|------------|
| {code} | {description} | {what to do} |

## Complete Example

**Request:**
```http
{METHOD} {/path} HTTP/1.1
Host: {host}
Authorization: Bearer {token}
Content-Type: application/json

{request body}
```

**Response:**
```json
{response body}
```

## Migration Notes
{Omit section if no changes from previous version}
- {Old behavior → New behavior}
- {Deprecated fields/params}
- {Version compatibility}
```

---

## F. Engineering Post — `engineering-post.md`
**Audience**: engineers (internal or external).
**Purpose**: deep dive into architecture, trade-offs, and lessons — goes far beyond restating the spec.

Only generate when there were notable technical decisions, performance work, or hard-won lessons.

```markdown
[agent: content]

# Building {Feature}: {Specific Technical Theme}

## Context
{The problem, constraints, and why this was non-trivial.
Include scale, timeline, existing system constraints.}

## Design Goals
- {Goal 1 — what we optimized for}
- {Goal 2}
- **Non-goal**: {what we explicitly chose NOT to optimize for, and why}

## Options We Considered

### Option A: {Name}
{Description, pros, cons}
**Verdict**: Rejected — {specific reason}

### Option B: {Name}
{Description, pros, cons}
**Verdict**: Chosen — {specific reason}

### Option C: {Name} (if applicable)
{Description, pros, cons}
**Verdict**: Rejected — {specific reason}

## Architecture & Implementation

{Components, data flow, key modules.
Reference specific files/modules if useful.}

> 💡 **Diagram opportunity**: {Describe the architecture or sequence diagram that would enhance this section}

### The Interesting Parts
{The non-obvious details — algorithms, optimizations, failure handling, edge cases.
This is where the post earns its keep. Be specific.}

### What Surprised Us
{Unexpected challenges, false assumptions, debugging stories.}

## Performance / Reliability Results
{Measured outcomes — only real numbers, no fake precision.
Latency, throughput, error rates, resource usage.
Include methodology if relevant.}

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| {metric} | {value} | {value} | {delta} |

## Lessons Learned
{Concrete lessons — what we'd do differently, what we'd do again.
Pull from `_context/lessons.md` if relevant entries exist.}

1. **{Lesson}**: {explanation}
2. **{Lesson}**: {explanation}
3. **{Lesson}**: {explanation}

## Follow-Up Work
{Only confirmed next steps — not generic roadmap.
Omit section if nothing is planned.}

- {Concrete follow-up with rationale}
```

**Rules**:
- Must go beyond restating the spec
- Prefer concrete trade-offs and debugging stories over abstract principles
- Include real numbers if they exist; if not, explicitly say "not yet measured"
- Reference specific files/modules where relevant

---

## G. Email Announcement — `email-announcement.md`
**Audience**: existing users, customers.
**Purpose**: direct communication about meaningful releases.

Only generate when the release is user-visible and requires awareness or action.

```markdown
[agent: content]

# Email Announcement — {Feature Title}

## Subject Line Options
1. {Option — outcome-focused, not "Introducing X"}
2. {Option — curiosity-driven}
3. {Option — benefit-led}

## Preheader
{1 sentence that complements the subject line — not a repeat}

## Email Body

Hi {audience},

{1-2 sentence opening: what shipped and why it matters to them specifically.}

### What's new
- **{Capability}**: {1-sentence benefit}
- **{Capability}**: {1-sentence benefit}
- **{Capability}**: {1-sentence benefit}

### Why this matters
{2-3 sentences — plain language, user perspective. What improves in their daily work.}

### What to do next
{Clear, single primary CTA.}

[**{CTA Button Text}** →]({link placeholder})

{Secondary action: "Read the full release notes →" or "Check the docs →"}

### If you need to upgrade
{Omit section if no action required.}
{Simple, specific upgrade instruction.}

Thanks,
{Team name}

---
*You're receiving this because {reason}. [Unsubscribe]({link})*
```
