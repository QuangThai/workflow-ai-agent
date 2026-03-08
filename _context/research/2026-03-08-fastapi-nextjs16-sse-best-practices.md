# FastAPI + Next.js 16 SSE Best Practices (Research Note)

> Date: 2026-03-08  
> Agent: brainstorm  
> Tag: [agent: brainstorm]  
> Scope: FastAPI backend + Next.js 16 frontend (App Router/BFF proxy)

## 1) Context loaded

- Product state confirms stack is FastAPI + Redis + Next.js 16 and current focus is upload UX/documentation, not realtime transport changes.
- No decision records currently present in `_context/decisions/`.
- PRDs require status/progress tracking (`FR-03` backend, `FR-DASH-004` + `FR-DETAIL-002` frontend), so SSE is aligned with existing requirements.
- Handoff queue includes active release/dev items unrelated to SSE; no blocking QA feedback specific to SSE architecture.

## 2) Current codebase findings (important)

Scopelytics already has a hybrid realtime implementation:

- **Backend SSE endpoint**: `scopelytics-ai-backend/src/app/api/v1/endpoints/analysis.py` exposes `GET /analysis/{analysis_id}/stream` using `StreamingResponse` + `text/event-stream`.
- **SSE format quality**: includes named events (`status`, `draft_features`, `final`), event IDs (`id:`), keepalive comments (`: keepalive`), `Last-Event-ID` parsing, and Redis Pub/Sub + DB fallback.
- **Frontend SSE consumer**: `scopelytics-ai-frontend/components/analysis/AnalysisWorkspace.StateManagement.tsx` uses `EventSource` with `withCredentials: true`, listens to custom event types, and falls back to polling when stream is unavailable.
- **BFF stream proxy**: `scopelytics-ai-frontend/app/api/proxy/[...path]/route.ts` detects `text/event-stream` and returns `backendResponse.body` directly (no buffering to arrayBuffer for SSE path).
- **WebSocket path also exists**: `scopelytics-ai-backend/src/app/api/v1/endpoints/ws.py` + `analysis_status_service.py`.

Conclusion: baseline architecture is already close to production best practice.

## 3) Problem definition

Need robust, low-latency progress streaming for long-running analysis jobs without introducing fragile infra complexity.

Pain points to prevent:

- Delayed/batched events due to proxy buffering.
- Lost progress when connection drops.
- Auth/session drift (especially with browser-native EventSource limitations).
- Resource leakage from stale long-lived connections.

## 4) Solution options (with trade-offs)

### Option A (recommended): SSE primary + polling fallback (current direction)

- Pros:
  - Simple browser support via `EventSource`.
  - One-way server->client fits analysis progress use case.
  - Works well with existing FastAPI + Next BFF model.
  - Already mostly implemented in repo.
- Cons:
  - Native `EventSource` cannot send custom auth headers.
  - Reconnect/replay semantics need explicit event IDs + resume policy.

### Option B: WebSocket primary

- Pros:
  - Full duplex, custom auth options.
  - Good for future interactive control channel.
- Cons:
  - More operational complexity (connection lifecycle, upgrades, infra tuning).
  - Overkill for mostly one-way status updates.

### Option C: Polling only

- Pros:
  - Very simple and predictable.
  - Easier to test in some environments.
- Cons:
  - Higher request overhead and slower perceived realtime UX.
  - Less efficient at scale than push streaming.

## 5) Best-practice guidance (FastAPI + Next.js 16)

### 5.1 Backend (FastAPI)

1. Keep stream response standards strict:
   - `Content-Type: text/event-stream`
   - `Cache-Control: no-cache`
   - `Connection: keep-alive`
   - `X-Accel-Buffering: no`
2. Keep connection alive with heartbeat comments every ~15-30s when idle (`: keepalive\n\n`).
3. Detect disconnect and stop work quickly (`await request.is_disconnected()` in stream loop).
4. Emit stable event IDs (`id:`) and support resume using `Last-Event-ID`.
5. Use shared event source (Redis Pub/Sub or durable store) so reconnects are not tied to a single process.
6. Use terminal event contract (`event: final`) so client can close stream deterministically.
7. Bound long-lived connections (timeout/cleanup/cancellation) to avoid fd/memory leaks.
8. Keep payloads compact and JSON-serializable; avoid large snapshots for each tick.

### 5.2 Frontend + BFF (Next.js 16)

1. Client component owns `EventSource` lifecycle; always close on unmount or when terminal status reached.
2. Keep polling fallback as safety net for environments where SSE fails.
3. In BFF proxy route, pass through stream body directly; do not buffer SSE response.
4. Preserve streaming headers (`content-type`, `cache-control`, `connection`, `x-accel-buffering`).
5. Use same-origin cookie auth when possible (works well with native `EventSource` + `withCredentials`).
6. If header-based bearer auth is required, use fetch-based SSE client library (not native `EventSource`).
7. Add jittered reconnect/backoff policy if using custom SSE client; avoid synchronized reconnect storms.

### 5.3 Infra/ops

1. Disable proxy buffering for SSE route (`X-Accel-Buffering: no` and/or nginx config).
2. Ensure idle timeouts (LB/proxy) exceed heartbeat interval.
3. Track metrics: active SSE connections, reconnect rate, stream error rate, time-to-first-event, terminal delivery success.
4. Log stream lifecycle with request/analysis IDs (avoid token leakage in logs).

## 6) Technical feasibility in Scopelytics

- **Feasibility**: High. Core SSE path is already implemented end-to-end.
- **Gap level**: Small-medium polish rather than net-new build.
- **Likely next hardening tasks**:
  - Add explicit replay buffer semantics for missed events (currently event ID exists, replay appears limited).
  - Add SSE-focused tests (backend stream contract + frontend fallback behavior).
  - Add stream observability dashboards/alerts.

## 7) Scope estimate and risk

- **If polishing current SSE path**: S-M.
- **If switching architecture to WS-primary**: M-L.

Main risks and mitigations:

- **Proxy buffering -> delayed UX**: enforce no-buffer headers and nginx/lb config checks.
- **Reconnect data loss**: event IDs + replay window/durable state.
- **Auth mismatch on reconnect**: cookie-based auth via BFF or fetch-based SSE for header auth.
- **Connection leaks**: cancellation + timeout + cleanup in `finally`.

## 8) Practical recommendation for this repo

Keep current design: **SSE-first + polling fallback**, and improve reliability/observability instead of replacing transport.

Priority order:

1. Lock stream contract (event schema, terminal semantics, replay expectations).
2. Add SSE-specific integration tests (disconnect/reconnect/terminal behavior).
3. Add operational telemetry and infra guardrails for buffering/timeouts.

## 9) Sources

- FastAPI docs - `StreamingResponse` and SSE tutorial (including keepalive, cache-control, buffering guidance).
- Next.js v16 docs - Route Handlers streaming patterns and self-hosted streaming/proxy buffering guidance.
- MDN - EventSource usage, `withCredentials`, event format, reconnect behavior.
