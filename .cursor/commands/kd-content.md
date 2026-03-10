Generate content artifacts from shipped features.

1. Load the `kd-content` skill
2. Follow the skill's full workflow (Step 1 → Step 5)
3. Pick up content tickets from `_handoff/queue/` where `to: content`
4. Load context from spec, product-state, and research notes
5. Generate: changelog entry, blog draft, social post, technical post (if applicable)
6. Save all drafts to `_context/content/YYYY-MM-DD-{slug}/`
7. Archive ticket and update spec status to `archived`
8. IMPORTANT: Content agent generates drafts ONLY — never modify files under `apps/`

Ticket to generate content for: $ARGUMENTS
