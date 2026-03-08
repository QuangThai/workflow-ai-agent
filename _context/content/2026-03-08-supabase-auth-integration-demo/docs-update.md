[agent: content]

# Documentation Update Draft (For Dev Pipeline)

## Summary
This draft outlines repository documentation updates required once implementation starts/finishes for `SPEC-002`.

## Proposed Docs Changes
1. Add `apps/service-b` auth architecture page:
   - Supabase browser/server client responsibilities
   - Proxy/middleware session refresh flow
   - Route protection behavior for SSR pages

2. Add `apps/service-a` API auth page:
   - Supabase JWT verification via JWKS
   - Guard/decorator conventions (`@Public`, route protection)
   - Expected claim mapping (`sub`, `role`, optional email)

3. Add operations runbook:
   - Redirect loop diagnosis
   - Invalid refresh token troubleshooting
   - JWKS cache/rotation handling
   - Callback and clock-skew checks

## Scope Restriction Note
This content draft does not modify `docs/` or `apps/` files directly. If documentation file updates are required in-repo, create a new dev handoff ticket.
