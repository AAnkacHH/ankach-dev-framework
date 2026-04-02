# Phase 6: Implementation — Comparison of 4 Frameworks

**Date:** 2026-03-29

---

## Key Patterns Found

### 1. Dispatch Models
- **Superpowers/Antigravity:** Fresh subagent per task, sequential, full task text injected
- **GSD:** Wave-based parallel, worktree-isolated, file paths passed (agent reads)
- **ECC Ralphinho:** DAG layers parallel, per-stage model routing, merge queue with eviction

### 2. Deviation Rules (GSD — most structured)
| Rule | Trigger | Action |
|------|---------|--------|
| Rule 1 (Bug) | Broken behavior | Auto-fix, max 3 attempts |
| Rule 2 (Missing Critical) | No validation, auth, error handling | Auto-add, max 3 attempts |
| Rule 3 (Blocking) | Missing deps, config | Auto-resolve, max 3 attempts |
| Rule 4 (Architectural) | New tables, major restructuring | **STOP — user decides** |

### 3. Status Codes (Superpowers/Antigravity)
- DONE — proceed to review
- DONE_WITH_CONCERNS — review concerns first
- BLOCKED — assess blocker, re-dispatch or escalate
- NEEDS_CONTEXT — provide context, re-dispatch

### 4. Key Insights
- **"Do not trust the report"** — reviewer reads actual code, not implementer's claims
- **De-sloppify pattern** (ECC) — two focused agents > one constrained agent
- **Analysis paralysis guard** (GSD) — 5+ reads without writes = STOP
- **Context isolation** — all frameworks agree: fresh context per task
- **Persistent debug state** (GSD) — debug files survive context resets

### 5. Git Strategy
- Atomic commits per task
- `--no-verify` for parallel agents, hooks validated post-wave
- Stage files individually, never `git add .`

### 6. Model Routing
- Mechanical tasks (CRUD, renaming): cheaper/faster model
- Integration tasks: standard model
- Architecture/review: most capable model
