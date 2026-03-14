Run structured, diff-aware code review for the active dev ticket.

Load the `kd-review` skill and follow its complete workflow.

1. Select active dev ticket and inspect changed scope
2. Classify findings by severity/category
3. Append review report with file:line and fix suggestions
4. Update gate fields: `review_status` + `review_findings_summary`
5. Block completion if critical/high findings remain

Ticket to review: $ARGUMENTS
