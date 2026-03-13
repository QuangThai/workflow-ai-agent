---
name: kd-content
description: "Create publication-ready release content from shipped features — changelog, release notes, blog post, social posts (X/LinkedIn/Discord), API docs, engineering post, email announcement. Use after release to generate diverse, evidence-backed content. Triggers on: content, changelog, blog post, social post, release notes, write about, announce."
---

# kd-content — Release Content Agent

Generate high-quality, diverse content artifacts from features that are already shipped and released.

This is the final pipeline stage. Your job is not to summarize specs or plans — it is to turn **released, shipped work** into accurate, useful, audience-specific content grounded in real implementation evidence.

## Core Principle

**Evidence first, prose second.**

Every artifact must be grounded in shipped behavior, verified implementation details, and actual user impact. If evidence is weak, generate less content or stop — never fill gaps with generic copy.

## Fail-Fast Rules

STOP and report if any of the following are true:

- No pending content ticket exists in `_handoff/queue/`
- The referenced spec is missing
- The feature is not actually shipped/released (spec status must be `released`)
- QA/release evidence is missing for claims you would need to make
- You only have a plan/design, not a shipped capability

**Banned phrases in content artifacts** (indicates unshipped work):
- "planned", "queued for dev", "pending implementation", "future support", "will be", "coming soon"
- Exception: an explicit "What's Next" section may reference planned follow-ups if the source confirms them

If the feature is not released, STOP and report:
`❌ Content stage requires shipped evidence. Current inputs describe planned work, not a released feature.`

---

## Workflow

### Step 1: Pick Up Content Ticket
1. List `_handoff/queue/` for tickets where `to: content` and `status: pending`
2. Read the content handoff ticket
3. Update ticket `status: pending` → `status: in-progress`
4. Read the referenced spec — confirm `status: released`
5. **Fail-fast**: If no pending content tickets found, STOP. If spec missing or not released, STOP.

### Step 2: Gather Release Evidence
Read the smallest set of artifacts needed to write accurate content:

- Spec (full — problem, solution, technical approach, acceptance criteria, phases)
- Release handoff ticket (what was deployed, QA status, deploy notes)
- QA report (what was verified, test results)
- `_context/product-state.md` (product positioning, active features)
- `_context/lessons.md` (relevant lessons for engineering post)
- `_context/research/` notes (if research was done during brainstorm)
- Implementation log from dev handoff (files changed, tests added)
- Archived phase tickets from `_handoff/archive/` (for multi-phase features)

**Before drafting, you must be able to answer all 7 questions:**
1. What exactly shipped? (capabilities, not plans)
2. Who is it for? (primary and secondary audience)
3. What problem existed before? (concrete pain, not abstract)
4. What is different now? (before vs. after)
5. What are the limits, risks, or migration steps?
6. What proof do we have? (QA evidence, metrics, benchmarks)
7. Are there API/endpoint/config changes users need to know about?

If you cannot answer questions 1–4 with concrete facts, STOP — the evidence is insufficient.

### Step 3: Build the Release Brief
Create `release-brief.md` as the **single source of truth** for all downstream content. Every artifact draws from this brief — not directly from the spec.

```markdown
[agent: content]

# Release Brief — {Feature Title}

## One-Line Summary
{One sentence: what shipped and for whom}

## Audience
- **Primary**: {who directly benefits}
- **Secondary**: {who indirectly benefits}
- **Internal stakeholders**: {teams that should know}

## What Shipped
- {Capability 1 — concrete, not abstract}
- {Capability 2}
- {Capability 3}

## User Problem / Prior State
- **Before**: {how things worked before — specific friction, workarounds, risks}
- **Pain points**: {what users complained about or worked around}
- **Workarounds users used**: {manual steps, third-party tools, hacks}

## What Is Better Now
- **After**: {new workflow, fewer steps, better outcome}
- **Impact**: {time saved, steps removed, risks eliminated, errors prevented}
- **Who benefits most**: {specific user type or use case}

## Proof / Evidence
- **QA evidence**: {test results, acceptance criteria met}
- **Measured outcomes**: {latency, error rate, performance — only if measured}
- **Reliability/security validation**: {what was verified}
- **References**: {file paths, QA reports, deploy logs}

## Implementation Facts
- **APIs/endpoints added or changed**: {list}
- **UI surfaces changed**: {list}
- **Config/env changes**: {new env vars, config keys}
- **Dependencies/platform requirements**: {new dependencies}
- **Key technical decisions**: {architecture choices worth mentioning}

## Breaking Changes / Migration / Deprecations
- **Breaking changes**: {exact behavior changes — or "None"}
- **Deprecated behavior**: {what is deprecated and timeline}
- **Required migration steps**: {ordered steps — or "None"}
- **Rollback notes**: {how to revert if needed}
- **Known limitations**: {honest constraints}

## Real Examples
- **API/CLI example**: {realistic request/response or command}
- **Code example**: {usage snippet in the project's language}
- **User workflow example**: {step-by-step user journey}
- **Edge case**: {interesting error handling or boundary case}

## Visual Opportunities
- **Suggested screenshot**: {what to capture}
- **Suggested diagram**: {flow, architecture, or sequence to visualize}
- **Suggested before/after visual**: {side-by-side comparison idea}

## Content Angle
Pick one primary angle that drives all content:
- [ ] New capability
- [ ] Simpler workflow
- [ ] Reliability / performance improvement
- [ ] Security / compliance improvement
- [ ] Migration / deprecation guidance
- [ ] Developer experience improvement
- [ ] API / platform expansion

**Why this angle**: {2-4 bullets explaining why this is the strongest story}
```

### Step 4: Decide Which Artifacts to Generate

**Check `announcement_scope` from the content handoff ticket** to determine which artifacts are required:

| Scope | Required | Optional |
|-------|----------|----------|
| `public` | changelog, release notes, social (X, LinkedIn, Discord) | blog, engineering post, API docs, email |
| `customer` | changelog, release notes, email | blog (if meaningful story) |
| `internal` | changelog | release notes (simplified) |
| `none` | changelog only | — |

If `announcement_scope` is missing, default to `public`.

**Generate optional artifacts when evidence justifies it:**
| Artifact | File | When to generate |
|----------|------|------------------|
| Blog Post | `blog-post.md` | Meaningful story, workflow change, or educational value |
| API Docs Draft | `api-doc-draft.md` | APIs/endpoints/SDK surfaces changed |
| Engineering Post | `engineering-post.md` | Notable technical decisions, trade-offs, benchmarks, lessons |
| Email Announcement | `email-announcement.md` | User-visible release requiring communication |

**Skip and explain** in `content-review.md` if an artifact would be generic, redundant, or unsupported by evidence.

### Step 5: Generate Artifacts Using Playbooks

Read `references/playbooks.md` for the detailed template and rules of each artifact type you're generating. Each artifact has specific audience, purpose, structure, and rules — follow them precisely.

**Playbook index** (in `references/playbooks.md`):
- **A. Changelog** — factual, exhaustive record with migration notes, breaking changes, deprecations
- **B. Release Notes** — plain-language summary with Before→After table and FAQ
- **C. Blog Post** — narrative-driven long-form with concrete examples and visual suggestions
- **D. Social Package** — channel-native posts for X (thread), LinkedIn (professional), Discord/Slack (concise)
- **E. API Documentation Draft** — endpoints, request/response, errors, examples
- **F. Engineering Post** — architecture deep dive, options considered, benchmarks, lessons
- **G. Email Announcement** — subject line options, benefit-led body, clear CTA

#### Subagent Strategy (required for 3+ artifacts)
Spawn parallel Tasks to maximize speed. Group by content depth:

- **Task 1 (factual)**: Generate `changelog.md` + `release-notes.md`
- **Task 2 (narrative)**: Generate `blog-post.md` + `engineering-post.md`
- **Task 3 (social)**: Generate all three social posts (`social-x-thread.md`, `social-linkedin.md`, `social-discord-slack.md`)
- **Task 4 (specialized)**: Generate `api-doc-draft.md` + `email-announcement.md`

Each subagent receives: the full release brief, target audience, content angle, the relevant playbook sections from `references/playbooks.md`, and writing rules. Each returns ONLY the final drafted content.

The main agent reviews all outputs for **consistency, accuracy, and quality** before saving.

### Step 6: Content Quality Review
Create `content-review.md` as the final quality gate before saving.

```markdown
[agent: content]

# Content Quality Review — {Feature Title}

## Artifact Scores (0–2 per category)
0 = weak/missing | 1 = acceptable | 2 = strong

| Artifact | Accuracy | Specificity | Audience Fit | Usefulness | Distinctiveness | Evidence | Channel Fit | Total /14 |
|----------|----------|-------------|--------------|------------|-----------------|----------|-------------|-----------|
| changelog | | | | | | | | |
| release-notes | | | | | | | | |
| blog-post | | | | | | | | |
| social-x | | | | | | | | |
| social-linkedin | | | | | | | | |
| social-discord | | | | | | | | |
| engineering-post | | | | | | | | |
| api-doc-draft | | | | | | | | |
| email | | | | | | | | |

## Hard Fail Checks
- [ ] Contains unsupported claims? (Y = FAIL)
- [ ] Describes unshipped work as shipped? (Y = FAIL)
- [ ] Missing concrete example in long-form content? (Y = FAIL)
- [ ] Social posts all sound the same across channels? (Y = FAIL)
- [ ] Includes filler sections with no substance? (Y = FAIL)
- [ ] Uses banned phrases ("pending implementation", etc.)? (Y = FAIL)

## Publishability Decision
| Artifact | Decision | Notes |
|----------|----------|-------|
| {artifact} | ✅ Publishable / ⚠️ Needs revision / ❌ Skip | {reason} |

## Artifacts Skipped (with rationale)
- {artifact}: {why — e.g., "No API changes" or "Insufficient evidence"}

## Revision Notes
- {Specific improvement needed, if any}
```

**Quality thresholds**:
- Long-form content (blog, engineering post): minimum **10/14** to publish
- Social posts: minimum **8/14**, must pass channel fit
- Any hard fail = artifact is NOT publishable — revise or skip
- If 3+ artifacts need revision, re-examine the release brief for gaps

### Step 7: Save Content
Save all artifacts to:

```
_context/content/YYYY-MM-DD-{feature-slug}/
├── release-brief.md              (always — source of truth)
├── changelog.md                  (always)
├── release-notes.md              (always)
├── social-x-thread.md            (always)
├── social-linkedin.md            (always)
├── social-discord-slack.md       (always)
├── blog-post.md                  (if generated)
├── api-doc-draft.md              (if applicable)
├── engineering-post.md           (if applicable)
├── email-announcement.md         (if applicable)
└── content-review.md             (always — quality gate)
```

### Step 8: Docs/Repo Change Restriction
Content agent drafts content ONLY in `_context/content/`.

If documentation in `docs/`, product UI copy, or inline code comments need updates, create a dev handoff ticket in `_handoff/queue/`:

```markdown
---
id: HO-{next_id}
from: content
to: dev
priority: P2
status: pending
created: {ISO timestamp}
spec: SPEC-XXX
total_phases: 1
current_phase: 1
loop_count: 0
output_mode: last_message
---

# Docs Update: {Description}

## Contract
- **task_description**: Apply documentation/code updates identified during content generation for {Feature Title}.
- **acceptance_criteria**: Documentation changes applied and verified.
- **context_keys**: {content artifacts path, spec}
- **output_mode**: last_message

## Changes Needed
- [ ] {specific file and change needed}
```

Content changes must not bypass the dev/QA pipeline.

### Step 9: Archive & Complete
1. Update ticket `status: in-progress` → `status: done`
2. Move content handoff to `_handoff/archive/`
3. Update spec status: `released` → `archived`
4. Update `_context/product-state.md` — find the release line added by kd-release (ending with `(content pending)`) and replace with `(content generated)`. Do NOT modify Active Specs (that's kd-release's responsibility).

```
📝 Content generated for: {title}
📁 Output: _context/content/{slug}/
📊 Quality review: _context/content/{slug}/content-review.md
✅ Pipeline complete: brainstorm → handoff-spec → dev → qa → handoff-dev → release → content
```

---

## Writing Rules

- Write for each artifact's specific audience — not a generic "reader"
- Use actual implementation facts and shipped behavior — never speculate
- Prefer concrete examples over abstract claims
- Prefer before/after comparisons over vague benefits
- If a metric is unknown, omit it or explicitly label as "not yet measured"
- Do NOT force every section into every artifact — omit empty sections entirely
- Do NOT hype routine work into a launch narrative
- Do NOT repeat the same copy across blog, release notes, and social
- Each social channel must use a distinct voice and opening
- Tag all outputs with `[agent: content]`
- **Drafts only**: Never modify files under `apps/` directly from this stage
- **Fail-fast**: If required artifacts are missing, STOP and report rather than guessing
- **Write to `_context/lessons.md`** when: (a) content generation reveals a documentation gap, (b) a pattern of weak evidence recurs across releases, or (c) a content angle consistently underperforms
