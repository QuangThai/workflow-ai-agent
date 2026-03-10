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

### Use react-doctor and vercel-react-best-practices for frontend work
- Context: User Profile frontend (ProfileForm, ProfileClient, AppHeader) was implemented without running react-doctor or consulting Vercel React Best Practices.
- Mistake: Skills were available but not invoked during implementation or QA.
- Correction: User asked why these skills were not used.
- Rule: When implementing or reviewing frontend (React/Next.js): (1) Run npx -y react-doctor@latest . --verbose --diff before marking done; (2) Reference vercel-react-best-practices skill for data fetching, rerenders, bundle size, and conditional rendering.
- Applies to: dev, qa
<!-- Agents: append new lessons below this line -->

### [2026-03-08] Prefer Reusable Templates Over Project Hardcoding
- **Context**: User requested making workflow docs reusable across different projects.
- **Mistake**: Root orchestrator docs and product-state content included hardcoded product/repo/stack names.
- **Correction**: Replace project-specific naming with neutral placeholders and multi-service patterns.
- **Rule**: Keep root workflow docs (`AGENTS.md`, `README.md`, `CLAUDE.md`, `_context/product-state.md`) project-agnostic unless explicitly asked to specialize.
- **Applies to**: all
