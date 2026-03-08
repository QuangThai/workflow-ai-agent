<!-- [agent: content] Generated from SPEC-005 -->

# Keeping PRDs and Code in Sync: How We Fixed Our Release Gate

When your release checklist says "run the API contract check" but the script fails before it even starts, something's wrong. We recently fixed exactly that — and closed a few other gaps between our documentation and implementation.

## The Problem

Our PRD review surfaced three issues:

1. **Broken contract check**: The verification script looked for a `backend` directory that didn't exist. Our actual backend lives in `scopelytics-ai-backend`. The script failed with `ENOENT`, so the release gate never ran.

2. **Incomplete coverage**: The script only verified auth, analysis, health, config, and feature-context endpoints. It ignored evaluations and feedback — endpoints the frontend actually calls. We were checking some contracts but not all.

3. **Config drift**: The frontend upload page expects `max_text_input_size`, `max_transcript_file_size_mb`, `allowed_transcript_formats`, and `transcript_upload_enabled` from the backend config API. The backend didn't return them. The frontend fell back to hardcoded defaults even when the config request succeeded — so validation rules could diverge between frontend and backend.

## The Solution

We made three targeted fixes:

- **Path fix**: Updated the contract script to use `scopelytics-ai-backend` as the backend root.
- **Full coverage**: Added evaluations and feedback routers to the verification script so every frontend proxy call is checked.
- **Config alignment**: Extended the backend `/config/upload` response with the four missing fields, mapped from existing settings. The frontend now receives backend validation rules when the config succeeds.

We also updated the Backend PRD to clarify which health endpoints serve ops (root-level) vs. the frontend proxy (API v1).

## Why It Matters

- **Release gate works**: `npm run check:api-contract` now passes in CI and locally.
- **Single source of truth**: Upload validation limits come from the backend instead of frontend defaults.
- **Documentation matches reality**: PRD and implementation are aligned.

## Try It Out

If you're running Scopelytics AI locally, run `npm run check:api-contract` from the frontend directory. You should see "API contract check passed" with the number of frontend calls verified. The upload page will now use backend-driven limits for text and transcript validation.
