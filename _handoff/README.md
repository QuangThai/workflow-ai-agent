# _handoff — Inter-Agent Queue

Handoff files pass work between pipeline stages.
Each file is a structured markdown document with metadata.

## Flow
```
queue/          → Agent picks up → Processes → Moves to archive/
```

## Naming Convention
```
YYYY-MM-DD_HH-MM_{from}-to-{to}_{title}.md
```

Example: `2026-03-06_14-30_brainstorm-to-dev_add-export-filters.md`

## Handoff Template
```markdown
---
id: HO-XXX
from: brainstorm|dev|qa|content
to: dev|qa|release|content
priority: P0|P1|P2
status: pending|in-progress|done|blocked
created: YYYY-MM-DDTHH:MM
spec: SPEC-XXX (if applicable)
---

# Title

## Context
What and why.

## Deliverables
- [ ] Checklist of what needs to be done

## Acceptance Criteria
- [ ] How to verify it's done

## References
- Links to _context/ docs, PRD sections, etc.
```
