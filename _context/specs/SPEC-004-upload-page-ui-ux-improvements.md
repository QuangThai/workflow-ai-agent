---
id: SPEC-004
title: Upload Page UI/UX Improvements
status: approved
priority: P1
effort: S
created: 2026-03-08
author: agent:brainstorm
---

# SPEC-004: Upload Page UI/UX Improvements

## Problem

Review of the `/upload` page against Vercel Web Interface Guidelines and ui-ux-pro-max best practices revealed several accessibility, touch-target, and UX consistency issues that affect usability—especially on mobile and for users relying on assistive tech.

## Proposed Solution

Targeted fixes grouped by priority. No redesign; minimal code impact.

---

## Phase 1 — Critical UX Fixes (P1)

### 1.1 ErrorAlert Dismiss Label Mismatch

**Problem:** In text mode, `ErrorAlert` shows "Choose another file" for the dismiss action, but the actual behavior is `onClearText`. The label is misleading.

**Fix:**
- Add optional `dismissLabel?: string` prop to `ErrorAlert`
- In `MediaModePanel`: pass `dismissLabel="Choose another file"` (or omit for default)
- In `TextModePanel`: pass `dismissLabel="Clear text"`
- Default to "Choose another file" when `dismissLabel` is not provided (backward compatible)

**Files:** `ErrorAlert.tsx`, `MediaModePanel.tsx`, `TextModePanel.tsx`

### 1.2 Touch Target Compliance

**Problem:** WCAG 2.5.5 recommends minimum 44×44px touch targets. Two controls fall short:
- `MetadataDisplay` toggle: `min-h-[36px]`
- `ModeToggle` button: `h-8` (32px)

**Fix:**
- `MetadataDisplay`: Change toggle to `min-h-[44px] min-w-[44px]` and ensure padding allows 44px hit area
- `ModeToggle`: Change button to `min-h-[44px]` (or use `size="lg"` / equivalent) while keeping compact visual style

**Files:** `MetadataDisplay.tsx`, `ModeToggle.tsx`

---

## Phase 2 — Layout & Polish (P2)

### 2.1 Card Bottom Margin

**Problem:** `FileUploader` Card has `mb-16` (4rem). On short viewports this creates excessive whitespace before the footer.

**Fix:**
- Reduce to `mb-8` or `mb-10` for better balance
- Or use responsive: `mb-8 sm:mb-12` if design requires more space on desktop

**Files:** `FileUploader.tsx`

### 2.2 Dropzone Format List Responsiveness

**Problem:** `DropzoneEmptyState` shows `Video: mp4, mov, webm, mkv · Audio: mp3, wav, m4a · Transcript: srt, vtt, txt` in one line. On narrow viewports this can overflow or wrap awkwardly.

**Fix:**
- Use `flex-wrap` and `gap-2` for the format spans
- Or truncate/abbreviate on small screens (e.g. "Video, Audio, Transcript" with tooltip)
- Ensure no horizontal scroll

**Files:** `DropzoneEmptyState.tsx`

---

## Phase 3 — Optional Enhancements (P3)

### 3.1 Mode Toggle Discoverability

**Problem:** The "Analyze text instead" / "Switch to media upload" control is a small ghost button. Users may not discover text-paste mode.

**Options (pick one if desired):**
- Add a subtle hint under the dropzone: "Or paste text for analysis"
- Make the toggle more prominent (e.g. segmented control or tabs)
- Add a "Paste text" link inside the empty dropzone state

**Scope:** Defer unless user feedback indicates low discovery.

---

## Technical Approach

### Backend Changes
- None

### Frontend Changes
- `ErrorAlert`: Add `dismissLabel` prop
- `MediaModePanel` / `TextModePanel`: Pass appropriate `dismissLabel`
- `MetadataDisplay`: Increase touch target to 44px
- `ModeToggle`: Increase touch target to 44px
- `FileUploader`: Adjust `mb-16` → `mb-8` or responsive
- `DropzoneEmptyState`: Improve format list layout for narrow viewports

## Acceptance Criteria

- [ ] In text mode, error dismiss button shows "Clear text" (not "Choose another file")
- [ ] MetadataDisplay "File details" toggle has ≥44×44px touch target
- [ ] ModeToggle "Analyze text instead" / "Switch to media upload" has ≥44×44px touch target
- [ ] Card bottom margin reduced; no excessive whitespace on 375px viewport
- [ ] Dropzone format list does not cause horizontal scroll on 375px
- [ ] All existing tests pass; `npm run lint && npm run build` succeeds

## Open Questions

- Confirm `mb-16` → `mb-8` with design/stakeholder if layout is intentional
- Phase 3 (mode discoverability): include in this spec or track separately?
