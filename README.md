# Agent-Driven Development Pipeline (Reusable Template)

> A local-first, file-based workflow system that orchestrates AI agents across the full product lifecycle — from brainstorming to content generation — using shared context and structured handoffs.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Pipeline Stages](#pipeline-stages)
- [Getting Started](#getting-started)
- [After Clone Customization](#after-clone-customization)
- [Copy-Paste System Prompt](#copy-paste-system-prompt)
- [Shared Context Layer](#shared-context-layer)
- [Handoff System](#handoff-system)
- [Usage Guide](#usage-guide)
- [Spec Lifecycle](#spec-lifecycle)
- [Error Handling & Feedback Loops](#error-handling--feedback-loops)
- [Project Structure](#project-structure)

---

## Overview

This workspace uses a **7-stage agent pipeline** to manage the full product development lifecycle. Each stage is implemented as an [Agent Skill](https://agentskills.io) (`.agents/skills/kd-*/SKILL.md`) with matching [Cursor slash commands](https://cursor.com/docs/agent/chat/commands) (`.cursor/commands/kd-*.md`). Skills read from a shared context layer and pass structured work items through a file-based handoff queue.

**Key design principles:**

- **Local-first** — All state lives in the filesystem. No external services required.
- **Context-sharing via symlinks** — The `_context/` folder is symlinked between the product decisions repo and the codebase, giving all agents a unified view of product state.
- **Structured handoffs** — Work moves between agents through markdown files with YAML frontmatter in `_handoff/queue/`, ensuring nothing is lost between sessions.
- **Human-in-the-loop** — Every critical transition (spec approval, deployment) requires explicit user confirmation.
- **Cross-agent compatible** — Skills follow the open [Agent Skills standard](https://agentskills.io/specification), working across Amp, Cursor, Claude Code, Codex, and more.

---

## Architecture

### High-Level Pipeline

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        AGENT PIPELINE                                   │
│                                                                         │
│  ┌──────────┐    ┌──────────────┐    ┌────────┐    ┌────────┐          │
│  │ BRAINSTORM├───►│ HANDOFF-SPEC ├───►│  DEV   ├───►│   QA   │          │
│  │ Research  │    │ Approve &    │    │ Build  │    │ Verify │          │
│  │ & Ideate  │    │ Queue        │    │ & Test │    │ & Gate │          │
│  └──────────┘    └──────────────┘    └────────┘    └───┬────┘          │
│                                                        │                │
│                                          FAIL ◄────────┤                │
│                                          (loop back)   │ PASS           │
│                                                        ▼                │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────────┐              │
│  │ CONTENT  │◄───┤   RELEASE    │◄───┤  HANDOFF-DEV     │              │
│  │ Blog,    │    │ Deploy &     │    │  Finalize &      │              │
│  │ Changelog│    │ Verify       │    │  Prepare Release │              │
│  └──────────┘    └──────────────┘    └──────────────────┘              │
└─────────────────────────────────────────────────────────────────────────┘
         ▲                    ▲                    ▲
         │                    │                    │
    ┌────┴────────────────────┴────────────────────┴────┐
    │              _context/ (Shared State)              │
    │  product-state.md │ specs/ │ decisions/ │ metrics/ │
    └───────────────────────────────────────────────────┘
```

### Data Flow

```
  User idea
    │
    ▼
  _context/specs/SPEC-XXX.md          ← brainstorm creates draft
    │
    ▼
  _handoff/queue/HO-XXX.md           ← handoff-spec creates ticket
    │
    ▼
  Codebase changes                    ← dev implements
    │
    ▼
  QA Report (in handoff ticket)       ← qa verifies
    │
    ▼
  _handoff/queue/HO-XXX-release.md   ← handoff-dev creates release ticket
    │
    ▼
  Production deployment               ← release deploys
    │
    ▼
  _context/content/YYYY-MM-DD-slug/  ← content generates artifacts
    │
    ▼
  _handoff/archive/                   ← everything archived
```

---

## Pipeline Stages


| #   | Stage            | Command            | Agent Role                                  | Input                | Output                              |
| --- | ---------------- | ------------------ | ------------------------------------------- | -------------------- | ----------------------------------- |
| 1   | **Discovery**    | `/kd-brainstorm`   | Research & ideate solutions                 | User idea or problem | Draft spec in `_context/specs/`     |
| 2   | **Approval**     | `/kd-handoff-spec` | Validate & queue for dev                    | Approved spec        | Handoff ticket in `_handoff/queue/` |
| 3   | **Development**  | `/kd-dev`          | Implement across one or more services       | Handoff ticket       | Code changes + tests                |
| 4   | **Quality**      | `/kd-qa`           | Run tests, lint, verify acceptance criteria | Completed dev work   | QA report (PASS/FAIL)               |
| 5   | **Finalization** | `/kd-handoff-dev`  | Prepare release package                     | QA-passed ticket     | Release handoff ticket              |
| 6   | **Release**      | `/kd-release`      | Deploy & verify production                  | Release ticket       | Live deployment                     |
| 7   | **Content**      | `/kd-content`      | Generate changelog, blog, docs              | Content ticket       | Content artifacts                   |

### Utility Commands

| Command | Description |
|---------|-------------|
| `/kd-fix` | Bug Fix Fast Path — skip brainstorm/spec, go straight to dev |
| `/kd-status` | Show pipeline status — scan queue, group by status/agent, flag at-risk tickets |

---

## Getting Started

### Prerequisites

- An AI coding agent that supports the [Agent Skills standard](https://agentskills.io) — for example [Amp](https://ampcode.com), [Cursor](https://cursor.com), [Claude Code](https://claude.ai), or [OpenAI Codex](https://openai.com/codex/)
- Required MCP servers installed and configured:
  - **Ref MCP** (documentation search/read): https://docs.ref.tools/install/index
  - **Exa MCP** (web + code search): https://exa.ai/docs/reference/exa-mcp
  - **Context7 MCP** (up-to-date library docs): https://context7.com/docs/installation
- Workspace with one or more project repos cloned:
  ```
  Workspace/
  └── apps/
      ├── service-a/             # Example: API/backend
      └── service-b/             # Example: web/frontend/mobile/worker
  ```

### MCP Quick Config

Use your MCP client config and add this shared `mcpServers` block:

```json
{
  "mcpServers": {
    "Ref": {
      "type": "http",
      "url": "https://api.ref.tools/mcp?apiKey=YOUR_REF_API_KEY"
    },
    "exa": {
      "type": "http",
      "url": "https://mcp.exa.ai/mcp"
    },
    "context7": {
      "url": "https://mcp.context7.com/mcp",
      "headers": {
        "CONTEXT7_API_KEY": "YOUR_CONTEXT7_API_KEY"
      }
    }
  }
}
```

Client notes:
- **Cursor**: add in MCP config (for example `~/.cursor/mcp.json`) or MCP settings UI.
- **OpenCode**: add the same `mcpServers` block in OpenCode MCP configuration.
- **Codex**: add the same `mcpServers` block in Codex MCP configuration.
- **Amp**: add the same `mcpServers` block in Amp MCP configuration.

Notes:
- For Ref, you can also send the key as header `x-ref-api-key` instead of query param.
- For Exa, if your setup requires auth, follow Exa docs to pass API key.

### Quick Start

**1. Start a new feature:**

```
/kd-brainstorm
> "I want to add real-time collaboration to the analysis page"
```

**2. Check what's in the pipeline:**

```
/kd-status
```

The agent scans `_handoff/queue/` and reports pending work by priority, flags at-risk tickets.

**3. Continue work on the next task:**

```
> "Pick up next task."
```

The agent picks the highest-priority pending ticket and loads the appropriate skill.

**4. Quick-fix a bug (skip brainstorm/spec):**

```
/kd-fix Login fails when email contains plus sign
```

**5. Run the full pipeline sequentially:**

```
/kd-brainstorm     →  Draft & approve a spec
/kd-handoff-spec   →  Queue it for development
/kd-dev            →  Implement the feature
/kd-qa             →  Verify quality
/kd-handoff-dev    →  Finalize for release
/kd-release        →  Deploy to production
/kd-content        →  Generate content artifacts
```

---

## After Clone Customization

When someone clones this workflow template, update these items first so it matches your real project:

### 1) Map your repositories to `apps/service-a` and `apps/service-b`

- `apps/service-a` should point to your backend/API repo (or your primary service repo).
- `apps/service-b` should point to your frontend/web/mobile repo (or your secondary service repo).
- If you have only one repo, keep `service-a` and note in docs that `service-b` is unused.
- If you have more than two repos, keep `service-a`/`service-b` as primary examples and document extra repos in `AGENTS.md`.

### 2) Add per-service conventions

For each service repo, create/update:
- `apps/service-a/AGENTS.md`
- `apps/service-a/PRD.md`
- `apps/service-b/AGENTS.md`
- `apps/service-b/PRD.md`

Include:
- language/framework conventions
- architecture boundaries
- quality gates (lint, typecheck, test)
- deploy/run commands
- required env vars

### 3) Update quality gates in skills

Adjust skill expectations to your stack:
- Python example gates (`ruff`, `mypy`, `pytest`) if service is Python
- Node/Frontend example gates (`npm run lint/build/test`) if service is JS/TS
- Replace defaults with your actual commands in service-level docs

### 4) Update deploy docs

- Replace `docs/deploy-project.sh` with your real deploy script and command.
- Add environment-specific release verification checklist.

### 5) Update product metadata placeholders

Edit `_context/product-state.md`:
- product name/domain
- repository naming
- deploy model

### 6) Validate pipeline wiring

Run a dry sequence with a small sample feature:
1. `/kd-brainstorm`
2. `/kd-handoff-spec`
3. `/kd-dev`
4. `/kd-qa`

Ensure each stage reads/writes `_context/` and `_handoff/` correctly.

---

## Copy-Paste System Prompt

Use this prompt after cloning to quickly align the agent with your project context:

```text
You are my workflow orchestrator for this repository.

Project profile:
- Product name: <YOUR_PRODUCT_NAME>
- Domain: <WHAT_IT_DOES_IN_ONE_LINE>
- Backend repo path: apps/service-a
- Frontend repo path: apps/service-b
- Tech stack (backend): <e.g., NestJS + Postgres>
- Tech stack (frontend): <e.g., Next.js 16>
- Deployment target: <e.g., Vercel + Fly.io>

Execution rules:
1) Always follow the 7-stage pipeline:
   /kd-brainstorm -> /kd-handoff-spec -> /kd-dev -> /kd-qa -> /kd-handoff-dev -> /kd-release -> /kd-content
2) Always read `_context/lessons.md` before starting stage work.
3) Enforce service-specific quality gates from:
   - apps/service-a/AGENTS.md and apps/service-a/PRD.md
   - apps/service-b/AGENTS.md and apps/service-b/PRD.md
4) Never mark work complete without runnable verification evidence.
5) Use `_handoff/queue/` + `_handoff/archive/` as the only handoff channel.
6) Keep changes minimal, reversible, and documented in handoff/spec files.

If required service files are missing, fail fast and report exactly what is missing.
```

---

## Shared Context Layer

The `_context/` directory is the **single source of truth** shared between all agents. It is designed to be symlinked between the product decisions repo and the codebase repos so that both Research Agents and Dev Agents operate on an identical state.

```
_context/
├── product-state.md              # Current priorities, active specs, quality metrics
├── specs/                        # Feature specifications
│   └── SPEC-XXX-feature-name.md  #   Lifecycle: draft → approved → implemented → released → archived
├── decisions/                    # Architecture & product decision records
│   └── YYYY-MM-DD-decision.md    #   Immutable once recorded
├── research/                     # Research notes, competitive analysis
│   └── YYYY-MM-DD-topic.md       #   Created by brainstorm agent
├── design/                       # Design docs, wireframes, UX decisions
│   └── DESIGN-XXX-title.md       #   Referenced by specs
├── metrics/                      # Quality metrics, KPIs, benchmarks
│   └── current-metrics.md        #   Updated by QA agent
└── content/                      # Generated content artifacts
    └── YYYY-MM-DD-feature-slug/  #   Created by content agent
        ├── changelog.md
        ├── blog-draft.md
        └── social-post.md
```

### Symlink Setup

To share context between repos, create a symlink from each repo's root:

```bash
# On Linux/macOS
ln -s /path/to/Workspace/_context /path/to/docs-repo/_context
ln -s /path/to/Workspace/_context /path/to/codebase-repo/_context

# On Windows (run as Administrator)
mklink /D "D:\docs-repo\_context" "D:\Workspace\_context"
mklink /D "D:\codebase-repo\_context" "D:\Workspace\_context"
```

### Context Rules

1. **Append-only** — Documents are never deleted; they move through lifecycle states and eventually get archived.
2. **Timestamped** — Every entry carries an ISO 8601 date for traceability.
3. **Agent-tagged** — Every entry records which agent created or modified it (e.g., `[agent: brainstorm]`).
4. **Status-tracked** — Specs follow a strict lifecycle: `draft → approved → implemented → released → archived`.

---

## Handoff System

The `_handoff/` directory is the **inter-agent work queue**. It uses structured markdown files with YAML frontmatter to pass work between pipeline stages.

```
_handoff/
├── queue/      # Active work items awaiting pickup
│   └── HO-XXX-{from}-to-{to}-{title}.md
└── archive/    # Completed work items (audit trail)
    └── (same naming convention)
```

### Handoff Ticket Structure

Every handoff ticket follows this schema:

```yaml
---
id: HO-001                           # Unique identifier
from: brainstorm                      # Originating stage
to: dev                               # Target stage
priority: P1                          # P0 (critical) | P1 (high) | P2 (normal)
status: pending                       # pending | in-progress | done | blocked
created: 2026-03-06T14:30            # ISO 8601 timestamp
spec: SPEC-001                        # Reference to source spec
---
```

### Routing Rules


| From         | To        | Trigger                                                    |
| ------------ | --------- | ---------------------------------------------------------- |
| `brainstorm` | `dev`     | Spec approved via `/kd-handoff-spec`                       |
| `dev`        | `qa`      | Implementation marked done (implicit — same ticket)        |
| `qa`         | `dev`     | QA fails — feedback loop with specific issues              |
| `dev`        | `release` | QA passes → `/kd-handoff-dev` creates release ticket       |
| `release`    | `content` | Deployment verified → `/kd-release` creates content ticket |


---

## Usage Guide

### Stage 1 — Brainstorm (`/kd-brainstorm`)

Start a product discovery session. The agent will:

1. Load current product state and past decisions from `_context/`
2. Build a Fact Ledger (Known Facts, Unknowns, Assumptions) before proposing solutions
3. Run mandatory parallel research tracks (codebase analysis + docs/best practices + external patterns when applicable)
4. Explore the problem space with 2-3 solution approaches backed by research findings
5. Produce a draft spec in `_context/specs/SPEC-XXX-title.md`
6. Present the draft for your review

```
/kd-brainstorm
> "We need batch export — users want to download all analyses at once"
```

**Output:** Draft spec with problem statement, proposed solution, technical approach, acceptance criteria, source-backed decisions, and linked research notes.

**Next step:** Review the spec. When satisfied, run `/kd-handoff-spec`.

---

### Stage 2 — Handoff Spec (`/kd-handoff-spec`)

Convert an approved spec into a dev-ready handoff ticket. The agent will:

1. Validate the spec has all required sections
2. Update spec status from `draft` → `approved`
3. Create a structured handoff ticket in `_handoff/queue/`
4. Update `_context/product-state.md` with the active spec

```
/kd-handoff-spec
```

**Output:** Handoff ticket with implementation plan, file paths, and acceptance criteria.

**Next step:** Run `/kd-dev` to begin implementation.

---

### Stage 3 — Development (`/kd-dev`)

Pick up the highest-priority handoff ticket and implement. The agent will:

1. Scan `_handoff/queue/` for pending `to: dev` tickets, sorted by priority
2. Load the spec, AGENTS.md conventions, and PRD requirements
3. Implement across the impacted services as needed
4. Run self-verification: lint, type-check, and relevant tests
5. Update the handoff ticket with an implementation log

```
/kd-dev
```

**Service conventions enforced (example for Python service):**

- Python 3.10+, async/await, full type hints
- `ruff check && ruff format` + `mypy src` + `pytest`

**Service conventions enforced (example for Node/Web service):**

- TypeScript strict mode, framework/client directives as required
- `npm run lint` + `npm run build` + `npm run test`

**Next step:** Run `/kd-qa` to verify the implementation.

---

### Stage 4 — Quality Assurance (`/kd-qa`)

Run comprehensive verification against the completed work. The agent will:

1. Execute all automated checks (lint, types, tests, contract verification)
2. Review changed files against AGENTS.md conventions
3. Verify each acceptance criterion from the spec
4. Generate a structured QA report with PASS/FAIL verdict

```
/kd-qa
```

**On PASS:** Move forward with `/kd-handoff-dev`.

**On FAIL:** The existing ticket is returned to dev — QA appends its report, increments `loop_count`, and resets `status: pending`. Run `/kd-dev` to address the specific issues in the QA feedback.

---

### Stage 5 — Finalize (`/kd-handoff-dev`)

Prepare QA-passed work for release. The agent will:

1. Verify the QA report shows PASS
2. Ensure PRD and AGENTS.md documentation is updated
3. Update spec status from `approved` → `implemented`
4. Create a release handoff ticket with deploy notes and rollback plan
5. Archive the original dev handoff ticket

```
/kd-handoff-dev
```

**Next step:** Run `/kd-release` to deploy.

---

### Stage 6 — Release (`/kd-release`)

Verify release status and update product state. This agent does **NOT** deploy code — deployment happens outside this workflow (CI/CD, manual deploy, `npm publish`, etc.). The agent will:

1. Pick up release tickets from `_handoff/queue/`
2. Ask the user whether the change is live to its intended audience
3. If not yet deployed: present readiness checklist and wait for user to deploy
4. If live: run post-release verification (health checks, smoke tests)
5. Update product state and archive the release ticket
6. Create a content handoff ticket for the shipped feature

```
/kd-release
```

**Next step:** Run `/kd-content` to generate content.

---

### Stage 7 — Content (`/kd-content`)

Generate content artifacts for the shipped feature. The agent will:

1. Read the content handoff ticket and original spec
2. Generate applicable content: changelog, blog post, social post, documentation updates
3. Save all artifacts to `_context/content/YYYY-MM-DD-feature-slug/`
4. Archive the content ticket and mark the spec as `archived`

```
/kd-content
```

**Output directory:**

```
_context/content/2026-03-06-batch-export/
├── changelog.md
├── blog-draft.md
├── social-post.md
└── docs-update.md
```

**Pipeline complete.**

---

## Spec Lifecycle

Every feature spec passes through a strict lifecycle, tracked in its YAML frontmatter:

```
  draft ──────► approved ──────► implemented ──────► released ──────► archived
    │               │                 │                  │                │
    │               │                 │                  │                │
 brainstorm    handoff-spec       handoff-dev        release           content
 creates it    approves it      code is done     deployed live     content done
```


| Status        | Set By             | Meaning                                    |
| ------------- | ------------------ | ------------------------------------------ |
| `draft`       | `/kd-brainstorm`   | Spec created, under review                 |
| `approved`    | `/kd-handoff-spec` | Spec approved, dev ticket queued           |
| `implemented` | `/kd-handoff-dev`  | Code complete, QA passed, ready for deploy |
| `released`    | `/kd-release`      | Live in production                         |
| `archived`    | `/kd-content`      | Content generated, lifecycle complete      |


---

## Error Handling & Feedback Loops

The pipeline includes a built-in feedback loop at the QA stage:

```
                    ┌──────────────────────────────┐
                    │                              │
                    ▼                              │
              ┌──────────┐     FAIL          ┌─────────┐
     ────────►│   DEV    ├──────────────────►│   QA    │
              │ Implement│                   │ Verify  │
              └──────────┘◄──────────────────┤         │
                    ▲           feedback      └────┬────┘
                    │           ticket             │
                    │                              │ PASS
                    │                              ▼
                    │                        ┌──────────┐
                    └────────────────────────┤HANDOFF-DEV│
                        (if issues found     └──────────┘
                         during release)
```

**QA Failure Flow:**

1. QA agent identifies specific issues with evidence
2. The **existing ticket is reused** — QA appends its report, increments `loop_count`, and resets `status: pending`
3. Dev agent picks up the same ticket on the next `/kd-dev` run and reads the appended QA feedback
4. Cycle repeats until QA passes (escalates to user at `loop_count >= 3`)

**Release Rollback Flow:**

1. If post-deploy verification fails, the release ticket contains rollback instructions
2. User executes rollback manually
3. Issues are fed back to dev via a new handoff ticket

---

## Project Structure

```
Workspace/
│
├── _context/                          # Shared context layer
│   ├── product-state.md              #   Current product state & priorities
│   ├── specs/                        #   Feature specifications (lifecycle-tracked)
│   ├── decisions/                    #   Architecture decision records
│   ├── research/                     #   Research notes & analysis
│   ├── design/                       #   Design docs & UX decisions
│   ├── metrics/                      #   Quality metrics & KPIs
│   └── content/                      #   Generated content artifacts
│
├── _handoff/                          # Inter-agent work queue
│   ├── queue/                        #   Active tickets awaiting pickup
│   └── archive/                      #   Completed tickets (audit trail)
│
├── .agents/                           # Agent Skills (SKILL.md w/ frontmatter)
│   └── skills/                       #   Pipeline skill definitions
│       ├── kd-brainstorm/            #     Stage 1: Discovery
│       ├── kd-handoff-spec/          #     Stage 2: Approval
│       ├── kd-dev/                   #     Stage 3: Development
│       ├── kd-qa/                    #     Stage 4: Quality Assurance
│       ├── kd-handoff-dev/           #     Stage 5: Finalization
│       ├── kd-release/              #     Stage 6: Release
│       └── kd-content/              #     Stage 7: Content
│
├── .cursor/                           # Cursor slash commands (plain Markdown)
│   └── commands/                     #   Invoke via /kd-* in Cursor chat
│       ├── kd-brainstorm.md          #     Trigger kd-brainstorm skill
│       ├── kd-handoff-spec.md        #     Trigger kd-handoff-spec skill
│       ├── kd-dev.md                 #     Trigger kd-dev skill
│       ├── kd-qa.md                  #     Trigger kd-qa skill
│       ├── kd-handoff-dev.md         #     Trigger kd-handoff-dev skill
│       ├── kd-release.md             #     Trigger kd-release skill
│       ├── kd-content.md             #     Trigger kd-content skill
│       ├── kd-fix.md                 #     Bug Fix Fast Path (utility)
│       └── kd-status.md              #     Pipeline status dashboard (utility)
│
├── docs/                              # Operational documentation
│   └── deploy-project.sh            #   Project deploy script (example name)
│
├── apps/                              # One or more code repositories
│   ├── service-a/                    #   Example: API service
│   │   ├── AGENTS.md                 #   Service-specific agent conventions
│   │   └── PRD.md                    #   Service requirements
│   └── service-b/                    #   Example: web/mobile/worker service
│       ├── AGENTS.md                 #   Service-specific agent conventions
│       └── PRD.md                    #   Service requirements
│
├── AGENTS.md                          # Root orchestrator configuration
└── README.md                          # ← You are here
```

---

## License

Adopt and customize for your project.
