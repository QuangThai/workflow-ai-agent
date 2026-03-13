# Release Handoff Ticket Template

Use this template when creating the release handoff ticket in `_handoff/queue/`.
The release agent asks the user about deploy status rather than assuming CI/CD — structure the ticket to support that flow.

```markdown
---
id: HO-{next_id}
from: dev
to: release
priority: {priority}
status: pending
created: {ISO timestamp}
spec: SPEC-XXX
total_phases: {total_phases}
current_phase: {total_phases}
loop_count: 0
origin_handoff_id: {ID of the first handoff for this spec}
service_scope: {list of affected services/repos}
risk_level: {low|medium|high}
rollback_plan: {short rollback strategy}
output_mode: last_message
---

# Release: {Feature Title}

## Contract
- **task_description**: Confirm release status for {Feature Title}. Verify whether the change is live to its intended audience, run post-release verification, update product state, and create content handoff.
- **acceptance_criteria**: Release verified (or readiness checklist presented if not yet deployed). Product state updated. Content handoff created.
- **context_keys**: {list of _context/ files — spec, QA report}
- **output_mode**: last_message

## Summary
{One-paragraph summary of what was built}

## Changes
{Group by service/repo — list key changes with file paths.
For multi-phase specs: summarize changes from ALL phases, not just the final one.
Read archived phase tickets from `_handoff/archive/` to build the complete picture.}

## QA Status
- All automated checks: ✅
- Acceptance criteria: ✅
- QA report: {location of QA report — in handoff ticket or separate file}

## Release Notes
- **Deploy order**: {which service(s) deploy first — or "no ordering required"}
- **Migration steps**: {ordered steps — or "None"}
- **Environment variables**: {new env vars with defaults — or "None"}
- **Feature flags**: {flag names, default states, rollout plan — or "None"}
- **Breaking changes**: {details — or "None"}
- **Rollback plan**: {exact commands/steps to revert}
- **Rollback trigger**: {specific conditions that warrant rollback}
- **PASS-WITH-NOTES observations**: {carry forward from QA — or "None"}

## Observability
- **Dashboards**: {links or names to monitor — or "N/A"}
- **Log queries**: {key queries to watch for errors — or "N/A"}
- **Alerts**: {any new alerts configured — or "N/A"}
- **Key metrics**: {what to watch and expected values}

## Post-Release Verification Checklist
- [ ] Health check passes
- [ ] Smoke test: {specific test}
- [ ] Monitor: {what to watch}
```
