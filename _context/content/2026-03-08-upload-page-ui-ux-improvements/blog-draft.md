<!-- [agent: content] Generated from SPEC-004 -->

# Upload Page UX Polish: Accessibility & Mobile-First Fixes

We ran our upload page through Vercel's Web Interface Guidelines and WCAG 2.5.5 — and shipped a handful of targeted fixes that make the experience better on mobile and for users relying on assistive tech.

## The Problem

Our upload flow supports two modes: drop a file (video, audio, transcript) or paste text for analysis. A UI/UX review surfaced:

- **Label mismatch**: In text mode, when an error occurred, the dismiss button said "Choose another file" — but the action was clearing the text. Confusing.
- **Touch targets**: Two controls (the "File details" toggle and the mode-switch button) were under the 44×44px minimum recommended by WCAG 2.5.5.
- **Layout**: The main card had 4rem bottom margin, creating excessive whitespace on short viewports.
- **Format list**: The supported formats line (Video: mp4, mov… · Audio: mp3… · Transcript: …) could overflow on 375px screens.

## The Solution

We applied minimal, targeted fixes — no redesign:

1. **ErrorAlert** now accepts an optional `dismissLabel` prop. In text mode we pass "Clear text"; in media mode we keep "Choose another file".
2. **MetadataDisplay** and **ModeToggle** buttons use `min-h-[44px] min-w-[44px]` (or equivalent) so they meet the touch target guideline.
3. **FileUploader** card margin: `mb-8` on mobile, `mb-12` on desktop.
4. **DropzoneEmptyState** format list uses `flex flex-wrap` with gap so it wraps cleanly on narrow viewports.

## How It Works

All changes are in the frontend upload components. No backend or API changes. The fixes follow existing patterns (Tailwind, shadcn/ui) and respect `prefers-reduced-motion` where applicable.

## Try It Out

Head to the upload page on a phone or resize your browser to 375px — the layout should feel tighter, and the format hints should wrap instead of overflowing. If you use text-paste mode and hit an error, the "Clear text" button now matches what it does.
