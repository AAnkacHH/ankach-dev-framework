---
name: workflow-feature
description: >
  Feature workflow for existing projects (brownfield). Runs all 9 phases sequentially
  with brownfield optimizations — shorter architecture, focused codebase research,
  skippable phases. Use when adding a feature, endpoint, or change to an existing project.
---

# Feature Workflow (Brownfield)

9-phase pipeline optimized for adding features to existing projects.
Each phase runs fully but with brownfield shortcuts where appropriate.

**Use for:** new endpoints, new entities, new services, bug fixes, refactoring in existing projects.
**For new projects or major features:** use `workflow-full` instead.

<HARD-GATE>
Do NOT skip phases 1, 3, 4, 5, 6, 7, 9. Each requires explicit user approval.
Phases 2 (Architecture) and 8 (Documentation) can be skipped with `skip` if user decides.
</HARD-GATE>

## Progress Tracking

After each phase, show:

```
Feature: {name} [brownfield]
Progress: ████░░░░░░░░░░░░░░░░ Phase {N}/9 — {Name}

  [x] Phase 1: Analysis
  [~] Phase 2: Architecture (skipped — no macro changes)
  [ ] Phase 3: Design           ← current
  [ ] Phase 4: Decomposition
  [ ] Phase 5: Validation
  [ ] Phase 6: Implementation
  [ ] Phase 7: Review
  [ ] Phase 8: Documentation
  [ ] Phase 9: Deploy
```

## User Commands

| Command | Effect |
|---------|--------|
| `approve` | Accept current phase, move to next |
| `reject` | Request changes, stay in current phase |
| `skip` | Mark phase as N/A (only Phase 2, 8) |
| `pause` | Save STATE.md, resume later |
| `abort` | Stop workflow, keep all artifacts |

---

# Phase 1: Analysis

Read `.claude/skills/analysis/SKILL.md` and execute it fully.

**Brownfield context:**
- Always brownfield — skip greenfield detection in Step 1
- Step 2 (Research): focus on codebase — existing patterns, affected layers, similar implementations
- External research only if new library or unfamiliar API involved

**On approve → show progress, continue to Phase 2.**

---

# Phase 2: Architecture

Read `.claude/skills/architecture/SKILL.md` and execute it fully.

**Brownfield context:**
- Most tech is already decided — focus only on NEW macro decisions
- If no new integrations, queues, protocols, or services → offer to skip

**Present to user:**
```
Phase 2: Architecture

The chosen approach from Phase 1 does not require new macro architecture decisions.
Existing stack and patterns apply.

`approve` to confirm and skip to Phase 3 | `reject` if there ARE macro decisions needed
```

If user confirms no changes → write brief confirmation doc → proceed to Phase 3.
If user says there ARE changes → run full architecture skill.

**On approve/skip → show progress, continue to Phase 3.**

---

# Phase 3: Design

Read `.claude/skills/design/SKILL.md` and execute it fully.

**Brownfield context:**
- Domain map: show only affected subdomains with impact (Major/Minor/None)
- Data model: only new/modified entities, reference existing patterns
- API contracts: only new/modified endpoints, follow existing response format

**On approve → show progress, continue to Phase 4.**

---

# Phase 4: Decomposition

Read `.claude/skills/decomposition/SKILL.md` and execute it fully.

**Brownfield context:**
- Sub-tasks reference exact existing file paths from codebase
- Wave plan: leverage independent stories for parallel agent execution
- Follow existing docs/requirements/ format if the project has one

**On approve → show progress, continue to Phase 5.**

---

# Phase 5: Validation

Read `.claude/skills/validation/SKILL.md` and execute it fully.

**Brownfield context:**
- Architecture review: check compatibility with existing stack, framework conventions
- Framework misuse checks: CLAUDE.md/AGENTS.md forbidden patterns
- If small feature (1-2 stories): validation can be brief but MUST still run

**On PASS + approve → show progress, continue to Phase 6.**
**On FAIL → back to Phase 3/4. Max 2 iterations.**

---

# Phase 6: Implementation

Read `.claude/skills/implementation/SKILL.md` and execute it fully.

**Brownfield context:**
- Create feature branch from main
- Baseline tests MUST pass before starting
- Follow existing codebase patterns exactly (Grep for examples before writing new code)
- Deviation rules: auto-fix Rules 1-3, STOP for Rule 4

**On approve → show progress, continue to Phase 7.**

---

# Phase 7: Review

Read `.claude/skills/review/SKILL.md` and execute it fully.

**Brownfield context:**
- Spec compliance: verify against Phase 3 design
- Code quality: check against project's CLAUDE.md/AGENTS.md conventions specifically
- Tests: verify no regression in existing tests + new tests cover new code

**On SHIP → show progress, continue to Phase 8.**
**On NEEDS WORK → back to Phase 6. Max 2 iterations.**
**On BLOCKED → back to Phase 4/5. Escalate.**

---

# Phase 8: Documentation

Read `.claude/skills/documentation/SKILL.md` and execute it fully.

**Brownfield context:**
- API docs: document only new/changed endpoints
- Dev notes: focus on non-obvious decisions and gotchas
- Changelog: brief entry (Added/Changed/Fixed)

**Skippable:** If user says `skip` → mark as N/A, proceed to Phase 9.

**On approve/skip → show progress, continue to Phase 9.**

---

# Phase 9: Deploy

Read `.claude/skills/deploy/SKILL.md` and execute it fully.

**Brownfield context:**
- Pre-deploy checklist with project-specific items
- Create PR with summary from all phases
- Archive `.planning/active/` → `docs/features/YYYY-MM-DD-{slug}/`

**On approve → DONE. Pipeline complete.**

---

## Resuming

If interrupted, say "continue" or run `/feature continue`.
Reads `.planning/STATE.md` → resumes from current phase.
