<!-- [agent: content] Generated from SPEC-004 -->

## [0.7.0] - 2026-03-08

### Changed
- **Upload page layout**: Reduced card bottom margin (mb-16 → mb-8 on mobile, mb-12 on desktop) for better balance on short viewports
- **Dropzone format list**: Format hints (Video, Audio, Transcript) now wrap correctly on narrow screens — no horizontal scroll on 375px viewport

### Fixed
- **Error alert in text mode**: Dismiss button now shows "Clear text" instead of "Choose another file" when pasting text for analysis — label matches the actual action
- **Touch targets (WCAG 2.5.5)**: "File details" toggle and "Analyze text instead" / "Switch to media upload" button now meet 44×44px minimum for better mobile and accessibility
