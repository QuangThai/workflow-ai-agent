---
name: kd-dev
description: "Pick up handoff tickets and implement features across backend and frontend. Use when ready to code approved specs. Triggers on: dev, implement, build, code."
---

# kd-dev — Development Agent

Pick up approved handoff tickets and implement features in the Scopelytics AI codebase.

## Workflow

### Step 1: Pick Up Work
1. List `_handoff/queue/` for pending tickets where `to: dev`
2. Sort by priority (P0 > P1 > P2)
3. Present to user: "Found {N} pending tickets. Starting with: {title} (P{X})"
4. Read the full handoff ticket

### Step 2: Load Context
1. Read the referenced spec from `_context/specs/`
2. Read relevant `AGENTS.md` files for code conventions
3. Read relevant `PRD.md` sections
4. Scan affected files listed in handoff ticket using `finder` / `Read`

### Step 3: Implement

#### Backend (scopelytics-ai-backend/)
Follow conventions from `AGENTS.md`:
- Python 3.10+, async/await, type hints everywhere
- FastAPI endpoints in `src/app/api/v1/`
- Pydantic schemas in `src/app/schemas/`
- SQLAlchemy models in `src/app/models/`
- Services in `src/app/services/`
- Alembic migrations if DB changes needed
- Tests alongside in `tests/`

#### Frontend (scopelytics-ai-frontend/)
Follow conventions from `AGENTS.md`:
- Next.js 16 App Router, React 19, TypeScript
- Components in `components/` with `"use client"` directive
- API clients in `lib/api.ts`
- Types in `lib/types.ts`
- Styling with Tailwind + `cn()` utility

### Step 4: Self-Verify
Run checks before marking done:

**Backend:**
```bash
cd scopelytics-ai-backend
ruff check --fix src && ruff format src
mypy src
pytest tests/{relevant_test_files} -v
```

**Frontend:**
```bash
cd scopelytics-ai-frontend
npm run lint
npm run build
npm run check:api-contract
```

### Step 5: Update Handoff
1. Update handoff ticket: `status: pending` → `status: in-progress` → `status: done`
2. Add implementation notes to the handoff ticket:
   ```markdown
   ## Implementation Log
   - Files changed: {list}
   - Tests added: {list}
   - Migration: {yes/no, migration name}
   - Notes: {anything QA should know}
   ```

### Step 6: Complete
Print summary:
```
✅ Implementation done: {title}
📁 Files changed: {count}
🧪 Tests: {pass/fail summary}
⏭️ Next: Run /kd-qa to verify
```

## Rules
- Always read existing code before modifying
- Make smallest reasonable diffs
- Follow existing patterns — don't introduce new libraries without approval
- Write tests for new functionality
- Update PRD.md if adding new endpoints/features
- Never skip linting or type checking
