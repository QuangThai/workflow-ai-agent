<!-- [agent: content] Generated from SPEC-005 -->

## [0.6.1] - 2026-03-08

### Added
- **Config API extended**: Backend `/config/upload` now returns `max_text_input_size`, `max_transcript_file_size_mb`, `allowed_transcript_formats`, and `transcript_upload_enabled`. The frontend upload page uses these values for validation instead of falling back to hardcoded defaults when the config succeeds.

### Fixed
- **API contract verification**: The `verify-api-contract` script now resolves to the correct backend directory (`scopelytics-ai-backend` instead of `backend`), so `npm run check:api-contract` runs successfully in CI and locally.
- **Contract coverage**: The verification script now includes evaluations and feedback endpoints, ensuring all frontend proxy calls are validated against backend routes.

### Changed
- **PRD documentation**: Backend PRD FR-08 now clarifies that root-level `/health`, `/ready`, `/metrics` serve load balancer and Prometheus, while the frontend proxy uses `/api/v1/health` and `/api/v1/health/dependencies` for programmatic checks.
