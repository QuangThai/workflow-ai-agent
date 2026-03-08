# Product State — Scopelytics AI

> Last updated: 2026-03-07
> Agent: content

## Product
- **Name**: Scopelytics AI (Meeting Video Analyzer)
- **Backend**: FastAPI + PostgreSQL + Redis + OpenAI (dual-LLM)
- **Frontend**: Next.js 16 + React 19 + TypeScript + Tailwind v4 + shadcn/ui
- **Deploy**: Docker Compose on GCP

## Current Focus
<!-- Updated by brainstorm/dev agents -->
- [ ] Document & clarify transcript upload validation rules (P1)
- [ ] Upload page UI/UX improvements (P1) — ErrorAlert label, touch targets, layout

## Recent Decisions
<!-- Auto-appended by kd-handoff-spec -->
- **2026-03-08** [SPEC-005] PRD Accuracy & API Contract: Fix verify-api-contract path (backend→scopelytics-ai-backend), add evaluations+feedback to contract check, extend /config/upload with transcript/text fields, clarify FR-08 health endpoints. P0/P1, effort S.
- **2026-03-08** [SPEC-004] Upload Page UI/UX: ErrorAlert dismissLabel for text mode, touch target 44px (MetadataDisplay, ModeToggle), Card mb-16→mb-8, Dropzone format list responsive. P1 priority, effort S.
- **2026-03-07** [SPEC-003] Auth Security Hardening: Fix password reset token lifecycle, login timing leak, CSRF strictness, rate limit hardening. P0 priority, effort M. Full OWASP audit-driven.
- **2026-03-07** [SPEC-002] AI Pipeline Latency: 5 targeted fixes — dedup token counting, tune recall threshold 50%→30%, reduce re-ask 2+2→1+1, cache classification LLM, eliminate preprocess double-count. P0 priority.
- **SPEC-001**: Defer format validation for `.txt` files (plain text, no timestamps required). Rationale: Flexibility for user-provided notes. AI layer will classify validity.

## Active Specs
<!-- Auto-appended by kd-handoff-spec, removed by kd-release -->
- ~~**SPEC-003** Auth Security Hardening — OWASP Audit (P0, M)~~ — `released` ✅ (content generated)
- ~~**SPEC-002** AI Analysis Pipeline Latency Optimization (P0, effort M)~~ — `implemented` ✅ (content generated)
- ~~**SPEC-005** PRD Accuracy & API Contract Fixes (P0/P1, S)~~ — `released` ✅ (content generated)
- SPEC-004: Upload Page UI/UX Improvements (P1, S) — handoff created
- SPEC-001: Clarify & Document Transcript Upload Validation Rules (P1, S)

## Quality Metrics
<!-- Updated by kd-qa -->
- Backend tests: `pytest` (see PRD.md §13)
- Frontend lint: `npm run lint && npm run build`
- API contract: `npm run check:api-contract`
