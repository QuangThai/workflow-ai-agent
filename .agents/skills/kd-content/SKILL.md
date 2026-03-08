---
name: kd-content
description: "Generate content from shipped features — changelogs, blog posts, social posts, docs. Use after release to create marketing and documentation content. Triggers on: content, changelog, blog post, write about."
---

# kd-content — Content Agent

Generate content artifacts from shipped features for Scopelytics AI.

## Workflow

### Step 1: Pick Up Content Ticket
1. List `_handoff/queue/` for tickets where `to: content` and `status: pending`
2. Read the content handoff ticket
3. Read the original spec for full context

### Step 2: Load Context
1. Read `_context/product-state.md` for product positioning
2. Read the spec's technical approach for accuracy
3. Check `_context/research/` for relevant research notes

### Step 3: Generate Content

Based on the content suggestions in the handoff, generate applicable items:

#### Changelog Entry
```markdown
## [{version}] - {date}

### Added
- {Feature description from user perspective}

### Changed
- {Any changes to existing behavior}

### Fixed
- {Any bug fixes included}
```

#### Blog/Social Post Draft
```markdown
# {Catchy Title}

{Hook — why should readers care?}

## The Problem
{What pain point this solves}

## The Solution
{How Scopelytics AI now handles this}

## How It Works
{Brief technical explanation — accessible to target audience}

## Try It Out
{Call to action}
```

#### Documentation Update
Update relevant docs in `docs/` or inline code documentation.

#### Technical Post (optional)
If there are interesting technical decisions:
```markdown
# Building {Feature}: {Technical Insight}

{What we learned, how we solved it, architecture decisions}
```

### Step 4: Save Content
Save generated content to `_context/content/`:
```
_context/content/YYYY-MM-DD-{feature-slug}/
├── changelog.md
├── blog-draft.md
├── social-post.md
└── docs-update.md
```

### Step 5: Archive & Complete
1. Move content handoff to `_handoff/archive/`
2. Update spec status: `released` → `archived`
3. Update `_context/product-state.md`

```
📝 Content generated for: {title}
📁 Output: _context/content/{slug}/
✅ Pipeline complete: brainstorm → spec → dev → qa → release → content
```

## Rules
- Write for the target audience (technical vs non-technical)
- Keep content accurate — reference actual implementation
- Don't over-hype — describe what was actually built
- Tag all content with `[agent: content]`
- This is the final stage — ensure everything is archived properly
