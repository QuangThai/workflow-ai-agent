<!-- [agent: content] Generated from SPEC-005 -->

**Short (Twitter/X):**
Fixed our API contract check so the release gate actually runs. Also aligned frontend upload validation with backend config — no more silent fallbacks. PRD and code now match. 🛠️

**Medium (LinkedIn):**
We fixed three gaps between our PRD and implementation: the API contract verification script now runs correctly, covers all frontend endpoints (including evaluations and feedback), and our upload config API returns the validation fields the frontend needs. Release gate works, and documentation matches reality.

**Long (Blog teaser):**
When your release checklist says "run the API contract check" but the script fails before it starts, you know something's off. We fixed that — plus aligned our frontend upload validation with backend config. Here's what we changed and why it matters.
