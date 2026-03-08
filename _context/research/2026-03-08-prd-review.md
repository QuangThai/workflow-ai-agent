# PRD Review — Backend & Frontend

> Date: 2026-03-08  
> Agent: brainstorm  
> Scope: scopelytics-ai-backend/PRD.md, scopelytics-ai-frontend/PRD.md  
> Research: Web search, mcp_web_fetch, subagent (standards & traceability)

## Executive Summary

Both PRDs are well-structured, provide clear traceability, and align with industry best practices. However, **several material discrepancies** exist between the documentation and the current implementation. These should be addressed to ensure accuracy and to support reliable release gating.

---

## 0. External Standards Validation (Research Findings)

Research was conducted using web search, documentation fetch, and subagent to validate the PRDs against industry standards and best practices.

### 0.1 Sources Consulted

| Source | Topic | URL |
|--------|-------|-----|
| Type.ai | PRD structure, components, best practices | https://blog.type.ai/post/how-to-write-a-product-requirements-document-prd-in-2024-with-examples-and-tips |
| River | PRDs engineering teams love, acceptance criteria, edge cases | https://rivereditor.com/guides/how-to-write-product-requirements-documents-2026 |
| IEEE 830 | Software Requirements Specification structure | https://standards.ieee.org/ieee/830/1221/ |
| ISO/IEC/IEEE 29148 | Requirements processes, lifecycle | https://www.iso.org/standard/72089.html |
| BesTest, TestRail, PractiTest | Requirements Traceability Matrix (RTM) | Multiple |
| OWASP Developer Guide | Security requirements in SDLC | https://devguide.owasp.org/ |

### 0.2 Industry Standards Checklist (IEEE 830 / ISO 29148)

| Element | Backend PRD | Frontend PRD | Standard |
|---------|-------------|--------------|----------|
| Document control / versioning | ✅ §1 | ✅ §1 | References & definitions |
| Problem statement | ✅ §3 | ✅ §2 (Product Objective) | Problem & context |
| Goals & non-goals | ✅ §4 | Implicit in §2 | Goals & success metrics |
| Functional requirements | ✅ §7 (FR-01..08) | ✅ §6 (FR-AUTH, FR-UPLOAD, etc.) | Functional specs |
| Non-functional requirements | ✅ §9 (NFR-01..04) | ✅ §7 (NFR-SEC, NFR-REL, etc.) | Quality attributes |
| Security requirements | ✅ §8 (SEC-01..04) | ✅ NFR-SEC | OWASP-aligned |
| Scope (in/out) | ✅ §6 | ✅ §4 | Boundary definition |
| Requirement-to-test traceability | ✅ §11 (RTM table) | ✅ §9 (Test Matrix) | Bidirectional RTM |
| Acceptance criteria | ✅ Per FR | ✅ Per FR (explicit) | Testable criteria |
| Risks & mitigations | ✅ §14 | ✅ §12 | Risk documentation |

**Verdict:** Both PRDs satisfy the core structural elements of IEEE 830 and ISO 29148.

### 0.3 Best Practices Assessment

| Best Practice | Backend | Frontend | Source |
|---------------|---------|----------|--------|
| Testable acceptance criteria | ✅ Specific, verifiable | ✅ Per FR | River, Type.ai |
| Edge cases documented | ⚠️ Partial (§14 risks) | ⚠️ Partial (§12 risks) | River |
| Ruthless prioritization (P0/P1/P2) | ⚠️ Not explicit in FRs | ⚠️ Not explicit | River |
| Bidirectional traceability | ✅ FR → test files | ✅ FR → TC-SM, TC-RG | BesTest, TestRail |
| Stable requirement IDs | ✅ FR-01, SEC-01, etc. | ✅ FR-AUTH-001, etc. | Xebrio, HHS |
| Security requirements explicit | ✅ SEC-01..04 | ✅ NFR-SEC | OWASP |
| Living document (version, date) | ✅ v1.1, 2026-03-05 | ✅ v1.1, 2026-03-05 | Type.ai |

### 0.4 Gaps vs. Best Practices

1. **Success metrics**: Neither PRD defines measurable KPIs (e.g., "80% of uploads complete in &lt;5 min"). Industry guidance recommends leading/secondary/primary metrics.
2. **Edge case coverage**: Could be expanded (e.g., "User navigates away mid-upload", "50% duplicate rows"). River emphasizes documenting these to reduce production bugs.
3. **Priority labels on requirements**: FRs do not carry P0/P1/P2. Adding them would support release trade-off decisions.

---

## 1. Backend PRD — Assessment

### ✅ Strengths

- **Standard structure**: Document Control, Executive Summary, Problem Statement, Goals/Non-Goals, Stakeholders, and Scope are clearly defined.
- **Strong traceability**: The Requirement-to-Test matrix (§11) maps FR/SEC/NFR to specific test files.
- **Testability-First Design Rules**: Detailed guidance on layered tests, dependency isolation, async conventions, and lifecycle testing.
- **Release Acceptance Criteria**: Concrete and executable (ruff, mypy, pytest).
- **Sources**: Internal and external references with links.

### ⚠️ Issues to Address

#### 1.1 FR-08: Operational Endpoints — Path Detail Missing

**Current (PRD §7 FR-08):**
> The system shall expose `/health`, `/ready`, and `/metrics`.

**Actual behavior:**
- Root-level (for load balancer/Prometheus): `/health`, `/ready`, `/metrics` — **correct**.
- API v1 (for frontend proxy): `/api/v1/health`, `/api/v1/health/dependencies` — **not documented in the PRD**.

**Recommendation:** Clarify that the frontend uses `/api/v1/health` and `/api/v1/health/dependencies` via the proxy, while root-level endpoints serve operational tooling.

#### 1.2 FR-02: Analysis Ingestion — "Media" Terminology

**PRD:** "The system shall accept `media`, `text`, `transcript`, and `json context` inputs."

**Implementation:** The backend accepts:
- Media: video/audio files (per `ALLOWED_VIDEO_FORMATS`, `ALLOWED_AUDIO_FORMATS`)
- Text: `text_content` (subject to `MAX_TEXT_CONTENT_LENGTH`)
- Transcript: `.txt`, `.srt`, `.vtt`, `.vrt` when `TRANSCRIPT_UPLOAD_ENABLED=true`
- JSON context: feature context file

**Recommendation:** Either keep as-is or clarify that "media" encompasses both video and audio.

#### 1.3 Traceability Table — Test Paths

The §11 table matches the current test layout. `test_health.py` covers `/api/v1/health`, `/api/v1/health/dependencies`, and `/metrics`.

---

## 2. Frontend PRD — Assessment

### ✅ Strengths

- **Route protection**: Clear description of public vs protected routes and `mva_access_token` cookie behavior.
- **Proxy prefixes**: Complete list (analysis, analysis-results, evaluations, health, config, feature-context, feedback).
- **Test Matrix**: Concrete smoke tests (TC-SM-01 through TC-SM-07) and regression tests (TC-RG-01 through TC-RG-07).
- **Release Gate**: Lint, build, contract check, smoke suite, and P0/P1 defect status.
- **Gaps section (§13)**: Acknowledges absence of automated test suite — transparent.

### ⚠️ Issues to Address

#### 2.1 API Contract Check — Incorrect Path, Script Fails

**PRD §8:** "Source of truth: `lib/api.ts` and `scripts/verify-api-contract.mjs`"

**Implementation:** The script uses:
```javascript
const backendRoot = path.resolve(frontendRoot, "..", "backend")
```
→ Resolves to `scopelytics/backend` (does not exist). The actual backend directory is `scopelytics-ai-backend`.

**Result:** `npm run check:api-contract` **fails** with `ENOENT`. The PRD states "Pass Criteria" for the contract check, but the script cannot run with the current repository layout.

**Recommendation:** Update the script:
```javascript
const backendRoot = path.resolve(frontendRoot, "..", "scopelytics-ai-backend")
```

#### 2.2 verify-api-contract — Missing Backend Endpoint Specs

**Implementation:** The script only parses:
- auth, analysis, health, config, feature_context

**Missing:**
- evaluations (analysis-results, evaluations/*)
- feedback (experience, summary, admin/list)

The frontend calls these endpoints via the proxy, but the script does not verify them because they are absent from `backendEndpointSpecs`. Add `evaluations.py` and `feedback.py` to the contract verification.

#### 2.3 Config API Contract — Backend Response Incomplete

**Frontend expects (lib/api.ts, useUploadConfig):**
- `max_text_input_size`
- `max_transcript_file_size_mb`
- `allowed_transcript_formats`
- `transcript_upload_enabled`

**Backend returns (config.py FileUploadConfigResponse):**
- `max_video_size_mb`, `max_audio_size_mb`, `max_json_context_file_size_mb`
- `allowed_video_formats`, `allowed_audio_formats`

**Impact:** The frontend falls back to defaults when these fields are missing (via `??`). The flow works, but the frontend is not aligned with backend validation rules. PRD FR-UPLOAD-002 states "with safe local defaults when config fails"; in practice, defaults are also used when the config **succeeds** but omits these fields.

**Recommendation:** Extend the backend `/config/upload` response to include:
- `max_text_input_size` (mapped from `MAX_TEXT_CONTENT_LENGTH`)
- `max_transcript_file_size_mb`
- `allowed_transcript_formats`
- `transcript_upload_enabled`

---

## 3. Cross-PRD Consistency

| Aspect | Backend PRD | Frontend PRD | Status |
|--------|-------------|--------------|--------|
| Auth endpoints | register, login, refresh, logout, me, forgot-password, reset-password | FR-AUTH-001..004 | ✅ Aligned |
| Upload modes | media, text, transcript, json context | FR-UPLOAD-001: media + text | ⚠️ Frontend does not explicitly treat transcript as a distinct mode |
| Export formats | CSV, JSON, XLSX | FR-DETAIL-004 | ✅ Aligned |
| Evaluation flows | run, list, stats, trends, calibration, retention | FR-EVAL-001..004 | ✅ Aligned |
| Health | /health, /ready, /metrics | proxy /health, /health/dependencies | ⚠️ Frontend uses /health/dependencies, not /ready |

---

## 4. Recommended Actions

### P0 (Blocking accuracy)

1. **Fix verify-api-contract.mjs**: Change `"backend"` to `"scopelytics-ai-backend"` so the script runs successfully.
2. **Extend backendEndpointSpecs**: Add evaluations and feedback routers to the contract check.

### P1 (Contract alignment)

3. **Backend /config/upload**: Add `max_text_input_size`, `max_transcript_file_size_mb`, `allowed_transcript_formats`, and `transcript_upload_enabled` to the response.
4. **Backend PRD FR-08**: Clarify root-level vs API v1 health endpoints.

### P2 (Documentation polish)

5. **Frontend PRD §4.3**: Document that the proxy forwards to `/api/v1/*` (via `BACKEND_API_URL`).
6. **Frontend PRD FR-UPLOAD-001**: Explicitly distinguish media vs transcript upload modes.

---

## 5. Conclusion

Both PRDs meet expectations for structure, traceability, and testability. The main issues are:

1. **Script path bug** preventing the API contract check from running.
2. **Config API contract gap** — backend does not expose all fields required by the frontend.
3. **verify-api-contract** does not cover evaluations and feedback endpoints.

After addressing P0 and P1 items, the PRDs will accurately reflect the implementation and the release gate will function as intended.
