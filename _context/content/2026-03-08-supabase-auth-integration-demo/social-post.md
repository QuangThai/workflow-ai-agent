[agent: content]

We prepared a practical auth foundation for Supabase + Next.js 16 + NestJS.

What’s included:
- SSR-safe session lifecycle with `@supabase/ssr`
- Proxy/middleware refresh flow to prevent cookie drift
- JWKS-based JWT verification strategy for protected NestJS APIs
- A 3-phase rollout plan with measurable acceptance criteria

Why it matters:
- Reduces redirect loops and session mismatch issues
- Improves backend auth enforcement consistency
- Creates a clean path for OAuth, RLS, and production hardening

Spec reference: `SPEC-002`
Status: approved and queued for dev implementation
