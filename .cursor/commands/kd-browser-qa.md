Run browser QA evidence capture for UI/web changes.

Load the `kd-browser-qa` skill and follow its workflow.

1. Identify changed UI routes/components from the ticket
2. Run Playwright first (`npx playwright test` scoped to affected flows)
3. If Playwright is unavailable, use manual browser fallback with explicit repro steps
4. Capture screenshots/log notes into `_context/metrics/qa/{HO-ID}/browser/`
5. Append Browser QA Report and update `evidence_paths`

Ticket to validate: $ARGUMENTS
