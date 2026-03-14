# _context — Shared Context Layer

This folder is the **single source of truth** shared between all agents (Brainstorm, Dev, QA, Release, Content).
It can be symlinked between the product decisions repo and the codebase repos.

## Structure

```
_context/
├── product-state.md        # Current product state, goals, priorities
├── lessons.md              # Cross-session agent memory (reviewed at session start)
├── specs/                  # Feature specs (lifecycle-tracked)
│   └── SPEC-XXX-title.md
├── decisions/              # Architecture & product decision records
│   └── YYYY-MM-DD-title.md
├── research/               # Research notes, competitive analysis
│   └── YYYY-MM-DD-topic.md
├── design/                 # Design docs, wireframes, UX decisions
│   └── DESIGN-XXX-title.md
├── metrics/                # KPIs, benchmarks, quality metrics
│   └── current-metrics.md
└── content/                # Generated content artifacts (per release)
    └── YYYY-MM-DD-slug/
        ├── release-brief.md
        ├── changelog.md
        ├── release-notes.md
        └── ...
```

## Rules
1. **Append-only** — never delete context files, only update status (e.g., `draft → approved → archived`)
2. **Timestamped** — every entry has ISO date
3. **Agent-tagged** — every entry has `[agent: brainstorm|dev|qa|release|content]` tag
4. **Status-tracked** — specs follow lifecycle: `draft → approved → implemented → archived`
