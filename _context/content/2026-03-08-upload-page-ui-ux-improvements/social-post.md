<!-- [agent: content] Generated from SPEC-004 -->

## LinkedIn / X Post

📱 Small UX polish on our upload page today.

Ran the upload flow against Vercel Web Interface Guidelines + WCAG — found a few gaps:

• Text mode error: dismiss button said "Choose another file" but the action was clearing text. Fixed.
• Touch targets: two controls were under 44px. Now compliant for mobile/accessibility.
• Layout: card had too much bottom margin on small screens. Tightened.
• Format list: "Video: mp4, mov… · Audio: mp3…" overflowed on 375px. Now wraps.

No redesign — targeted fixes. Better for anyone on a phone or using assistive tech.

#ux #accessibility #wcag #nextjs
