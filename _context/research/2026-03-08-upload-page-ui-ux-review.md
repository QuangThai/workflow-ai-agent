# Research: Upload Page UI/UX Review

> Date: 2026-03-08 | Topic: /upload page UI/UX | [agent: brainstorm]

## Sources Consulted

- Vercel Web Interface Guidelines (https://github.com/vercel-labs/web-interface-guidelines)
- ui-ux-pro-max skill (touch targets, accessibility, layout)
- Scopelytics frontend PRD §FR-UPLOAD
- Codebase: `scopelytics-ai-frontend/app/upload/`, `components/upload/`

## Fact Ledger Updates

### Verified Facts
- Page uses AppLayout, FileUploader, ModeToggle (media/text), MediaModePanel, TextModePanel
- Step indicator: Select → Validate → Analyze → Results (4 steps)
- FR-UPLOAD-005: Upload and finalization progress must be clearly shown ✓
- MotionConfig with prefersReducedMotion ✓
- ProgressBar/TextProgressBar use aria-live="polite" ✓

### Issues Identified
1. **ErrorAlert** — Dismiss button label hardcoded "Choose another file"; in text mode action is clear text → wrong label
2. **MetadataDisplay** — Toggle button min-h-[36px] < 44px touch target (WCAG 2.5.5)
3. **ModeToggle** — Button h-8 (32px) < 44px touch target
4. **ContextSection** — Toggle is `<button>` without explicit type="button" (safe by default but worth noting)
5. **DropzoneEmptyState** — Format list (Video: mp4, mov… · Audio: mp3… · Transcript: …) may overflow on narrow viewports
6. **Card** — mb-16 creates large bottom gap; may push footer far down on short viewports

## Best Practices Applied

| Rule | Status |
|------|--------|
| Touch target ≥44px | Partial — MetadataDisplay, ModeToggle below |
| aria-label on icon buttons | ✓ FileCard, StepIndicator |
| aria-live for async updates | ✓ ProgressBar |
| focus-visible:ring | ✓ ContextSection, dropzone |
| Labels for form controls | ✓ TextInput, ContextSection (sr-only where appropriate) |
| prefers-reduced-motion | ✓ MotionConfig in UploadMeetingClient |
| Loading states end with … | ✓ "Uploading…", "Processing…", "Submitting…" |

## Recommendation

Prioritize fixes by impact:
1. **P1** — ErrorAlert dismiss label (wrong UX in text mode)
2. **P2** — Touch target compliance (MetadataDisplay, ModeToggle)
3. **P3** — Layout polish (mb-16, format list responsive)
4. **P4** — Mode toggle discoverability (optional enhancement)
