# Lessons Learned — Cross-Session Agent Memory

> Review this file at the start of every session. Update it after every correction from the user.

## How to Use This File

1. **At session start**: Scan for patterns relevant to the current task.
2. **After any user correction**: Add a new entry with the pattern, the mistake, and the rule to prevent it.
3. **Periodically**: Review and consolidate — merge related lessons, remove outdated ones.

## Format

```markdown
### [YYYY-MM-DD] Lesson Title
- **Context**: What task was being done
- **Mistake**: What went wrong
- **Correction**: What the user said
- **Rule**: The rule to follow going forward
- **Applies to**: brainstorm | dev | qa | all
```

---

## Lessons

<!-- Agents: append new lessons below this line -->

### [2026-03-08] Prefer Reusable Templates Over Project Hardcoding
- **Context**: User requested making workflow docs reusable across different projects.
- **Mistake**: Root orchestrator docs and product-state content included hardcoded product/repo/stack names.
- **Correction**: Replace project-specific naming with neutral placeholders and multi-service patterns.
- **Rule**: Keep root workflow docs (`AGENTS.md`, `README.md`, `CLAUDE.md`, `_context/product-state.md`) project-agnostic unless explicitly asked to specialize.
- **Applies to**: all
