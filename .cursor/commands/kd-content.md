Generate publication-ready content artifacts from implemented, QA-verified features.

Load the `kd-content` skill and follow its complete workflow.

1. Pick up content tickets from `_handoff/queue/` where `to: content` and `status: pending`
2. Gather completion evidence: spec, QA report, finalization ticket, product state
3. Build a Completion Brief as the single source of truth
4. Generate artifacts based on `announcement_scope`: changelog, release notes, social posts, blog, etc.
5. Run content quality review and score each artifact
6. Save all drafts to `_context/content/YYYY-MM-DD-{slug}/`
7. Archive ticket, update spec status to `archived`

Content agent generates drafts ONLY — never modify files under `apps/`.

Ticket to generate content for: $ARGUMENTS
