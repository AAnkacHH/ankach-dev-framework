---
name: feature
description: >
  End-to-end feature workflow for existing projects (brownfield). Walks through all 9 phases
  sequentially: analyze → architect → design → decompose → validate → build → review → document → deploy.
  Each phase has a manual approval gate. Use when adding a feature, endpoint, or module to an existing project.
---

# Feature Workflow (Brownfield)

End-to-end pipeline for adding a feature to an existing project.
Walks through all 9 phases sequentially with manual approval at each gate.

<HARD-GATE>
This workflow follows ALL 9 phases in order. Do NOT skip any phase.
Each phase requires explicit user approval before proceeding to the next.
</HARD-GATE>

## How This Works

```
USER: /feature Add bulk shipment endpoint
  │
  ▼
Phase 1: Analysis ──── approve ────┐
Phase 2: Architecture ── approve ──┤
Phase 3: Design ──────── approve ──┤  All sequential,
Phase 4: Decomposition ─ approve ──┤  auto-transition
Phase 5: Validation ──── approve ──┤  on each approve
Phase 6: Implementation  approve ──┤
Phase 7: Review ──────── approve ──┤
Phase 8: Documentation ─ approve ──┤
Phase 9: Deploy ──────── approve ──┘
                            │
                         DONE ─── archived to docs/features/
```

**User controls:** `approve` / `reject` / `skip` (skip = mark phase as N/A and continue)

---

## Phase 1: Analysis

Read and follow `.claude/skills/analysis/SKILL.md`.

**Brownfield optimizations:**
- Step 1 (Context): always brownfield — skip greenfield detection
- Step 2 (Research): focus on codebase patterns, existing code in affected area
- Step 3 (Questions): skip questions answerable from codebase

**On approve:** announce "Phase 1 complete. Moving to Phase 2: Architecture."

---

## Phase 2: Architecture

Read and follow `.claude/skills/architecture/SKILL.md`.

**Brownfield optimizations:**
- Most tech is already decided — focus only on NEW decisions
- If no new macro decisions needed, write brief "Architecture Confirmation" and proceed
- Gray areas: only for new integrations, new queues, new protocols

**Shortcut:** If user says "no architecture changes needed" → write minimal confirmation doc, proceed to Phase 3.

**On approve:** announce "Phase 2 complete. Moving to Phase 3: Design."

---

## Phase 3: Design

Read and follow `.claude/skills/design/SKILL.md`.

**Brownfield optimizations:**
- Domain map: focus on affected subdomains only, show impact (Major/Minor/None)
- Data model: only new/modified entities
- API contracts: only new/modified endpoints

**On approve:** announce "Phase 3 complete. Moving to Phase 4: Decomposition."

---

## Phase 4: Decomposition

Read and follow `.claude/skills/decomposition/SKILL.md`.

**Brownfield optimizations:**
- Sub-tasks reference exact file paths from existing codebase
- Wave plan leverages independent stories for parallel execution
- Stories reference existing patterns: "Similar to Story X in Epic Y"

**On approve:** announce "Phase 4 complete. Moving to Phase 5: Validation."

---

## Phase 5: Validation

Read and follow `.claude/skills/validation/SKILL.md`.

**Brownfield optimizations:**
- Architecture review: check compatibility with existing stack
- Framework misuse checks based on project's CLAUDE.md/AGENTS.md rules
- Requirements coverage: verify against Phase 3 design

**On PASS + approve:** announce "Validation passed. Moving to Phase 6: Implementation."
**On FAIL:** loop back to Phase 3 or 4 as needed. Max 2 iterations.

---

## Phase 6: Implementation

Read and follow `.claude/skills/implementation/SKILL.md`.

**Execution:**
- Create feature branch
- Execute waves sequentially, stories within wave in parallel
- Report after each wave, user approves next wave
- Use `implementer` agent per story

**On approve:** announce "Implementation complete. Moving to Phase 7: Review."

---

## Phase 7: Review

Read and follow `.claude/skills/review/SKILL.md`.

**Staged order:**
1. Spec compliance (read CODE, not reports)
2. Code quality (only if spec passes)
3. Test verification

**On SHIP:** announce "Review passed: SHIP. Moving to Phase 8: Documentation."
**On NEEDS WORK:** loop back to Phase 6. Max 2 iterations.
**On BLOCKED:** loop back to Phase 4/5. Escalate to user.

---

## Phase 8: Documentation

Read and follow `.claude/skills/documentation/SKILL.md`.

**Generate from code (source of truth):**
- API docs for new/changed endpoints
- Dev notes: how it works, gotchas
- Changelog entry

**On approve:** announce "Documentation complete. Moving to Phase 9: Deploy."

---

## Phase 9: Deploy

Read and follow `.claude/skills/deploy/SKILL.md`.

**Steps:**
1. Pre-deploy checklist
2. Migration plan (if DB changes)
3. Create PR or merge
4. Post-deploy verification
5. Archive `.planning/active/` → `docs/features/YYYY-MM-DD-{slug}/`

**On approve:** announce "Feature complete. Pipeline finished."

---

## Progress Tracking

After each phase completion, update `.planning/STATE.md` and show progress:

```
Feature: {name}
Progress: ████████░░░░░░░░░░░░ Phase {N}/9 — {Phase Name}

Completed:
  [x] Phase 1: Analysis
  [x] Phase 2: Architecture
  [x] Phase 3: Design
  [ ] Phase 4: Decomposition ← current
  [ ] Phase 5: Validation
  [ ] Phase 6: Implementation
  [ ] Phase 7: Review
  [ ] Phase 8: Documentation
  [ ] Phase 9: Deploy
```

---

## User Commands During Workflow

| Command | Effect |
|---------|--------|
| `approve` | Accept current phase, move to next |
| `reject` | Request changes, stay in current phase |
| `skip` | Mark phase as N/A, move to next (only for Phases 2, 8) |
| `pause` | Save state, can resume later with `/feature continue` |
| `abort` | Stop workflow, keep all artifacts created so far |

**Skippable phases:**
- Phase 2 (Architecture) — if no macro changes needed
- Phase 8 (Documentation) — if user explicitly opts out

**Non-skippable phases:** 1, 3, 4, 5, 6, 7, 9

---

## Resuming an Interrupted Workflow

If the session is interrupted or user says `/feature continue`:

1. Read `.planning/STATE.md` — find current phase
2. Read all completed phase artifacts
3. Resume from the current phase
4. Show progress tracker

---

## Anti-Rationalization Table

| Excuse | Reality |
|--------|---------|
| "This feature is small, just skip to implementation" | Small features with hidden complexity cause the most rework. Follow the phases. |
| "Architecture hasn't changed" | Confirm it. Write the 2-line confirmation. 30 seconds vs hours of wrong assumptions. |
| "Let me skip validation, I'll catch issues in review" | Validation catches design issues. Review catches code issues. Different things. |
| "Documentation can wait" | Documentation written after deploy is never written. Do it now. |
| "Just merge it, deploy is simple" | Pre-deploy checklist exists because 'simple deploys' cause production incidents. |
