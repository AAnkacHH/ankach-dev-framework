---
name: workflow-full
description: >
  Full 9-phase development workflow for new projects or major features.
  Greenfield + brownfield. Runs all phases sequentially with approval gates.
  Use when starting a new project, new module, or major cross-cutting feature.
---

# Full Development Workflow

Complete 9-phase pipeline. Each phase runs fully, each has a manual approval gate.
Auto-transitions to the next phase after each `approve`.

**Use for:** new projects, new modules, major refactoring, cross-cutting features.
**For smaller features on existing projects:** use `workflow-feature` instead.

<HARD-GATE>
Do NOT skip any phase. Each phase requires explicit user approval.
On `reject` at any phase — fix and re-present. On `abort` — stop, keep all artifacts.
</HARD-GATE>

## Progress Tracking

After each phase, show:

```
Feature: {name}
Progress: ████░░░░░░░░░░░░░░░░ Phase {N}/9 — {Name}

  [x] Phase 1: Analysis
  [x] Phase 2: Architecture
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
| `pause` | Save STATE.md, resume later |
| `abort` | Stop workflow, keep all artifacts |

---

# Phase 1: Analysis

Read `.claude/skills/analysis/SKILL.md` and execute it fully — all 6 steps.

**Summary of steps:**
1. Context (silent) — read CLAUDE.md, AGENTS.md, .context/, detect brownfield/greenfield
2. Research — brownfield: codebase patterns; greenfield: external research via researcher agent
3. Questions — one at a time, max 7, multiple choice preferred
4. Understanding Lock — **HARD GATE** — summary → user confirms
5. Approaches — 2-3 with trade-offs
6. Approval + Document — **HARD GATE** — write to `.planning/active/01-analysis/`

**On approve → show progress, continue to Phase 2.**

---

# Phase 2: Architecture

Read `.claude/skills/architecture/SKILL.md` and execute it fully — all 6 steps.

**Summary of steps:**
1. Context (silent) — read Phase 1 analysis
2. Gray Areas — find ambiguities in macro decisions
3. Questions — tech decisions, "3 Questions" gate per pattern
4. Architecture Decisions — ADR for each choice
5. System Design Checklist — functional, non-functional, technical, operations
6. Approval + Document — **HARD GATE** — write to `.planning/active/02-architecture/`

**On approve → show progress, continue to Phase 3.**

---

# Phase 3: Design

Read `.claude/skills/design/SKILL.md` and execute it fully — all 6 steps.

**Summary of steps:**
1. Context (silent) — read Phase 2 architecture
2. Domain Map — subdomains, bounded contexts, relationships, impact
3. Data Model — entities, fields, relations, migrations
4. API Contracts — endpoints, DTOs, validation, status codes
5. Self-Review — coverage, consistency, no placeholders
6. Approval + Document — **HARD GATE** — write to `.planning/active/03-design/`

**On approve → show progress, continue to Phase 4.**

---

# Phase 4: Decomposition

Read `.claude/skills/decomposition/SKILL.md` and execute it fully — all 6 steps.

**Summary of steps:**
1. Context (silent) — read Phase 3 design
2. Epic Definition — goal, scope, acceptance criteria
3. Story Decomposition — stories with Depends on + Wave + sub-tasks
4. Sub-task Breakdown — exact files, classes, actions for AI delegation
5. Wave Plan — parallel groups within stories
6. Approval + Document — **HARD GATE** — write to `.planning/active/04-decomposition/`

**On approve → show progress, continue to Phase 5.**

---

# Phase 5: Validation

Read `.claude/skills/validation/SKILL.md` and execute it fully — all 6 steps.

**Summary of steps:**
1. Context (silent) — read Phase 2 + 3 + 4
2. Architecture Review — deprecated tech, anti-patterns, security, performance, framework misuse
3. Requirements Coverage — every Phase 3 requirement → Phase 4 story
4. Story Completeness + Alignment + Scope + Placeholders
5. Validation Report — PASS/FAIL with issues list
6. Approval — **HARD GATE** — PASS → Phase 6, FAIL → back to Phase 3/4 (max 2 iterations)

**On PASS + approve → show progress, continue to Phase 6.**

---

# Phase 6: Implementation

Read `.claude/skills/implementation/SKILL.md` and execute it fully — all 6 steps.

**Summary of steps:**
1. Context (silent) — read Phase 4 wave-plan + Phase 5 validation
2. Setup — git branch, verify baseline tests pass
3. Execute Waves — dispatch `implementer` agents per wave, deviation rules
4. Wave Report — **GATE per wave** — show progress, user approves next wave
5. Implementation Summary — all waves done
6. Approval — **HARD GATE** — write to `.planning/active/06-implementation/`

**On approve → show progress, continue to Phase 7.**

---

# Phase 7: Review

Read `.claude/skills/review/SKILL.md` and execute it fully — all 6 steps.

**Summary of steps:**
1. Context — read Phase 6 reports + actual code (read CODE, not reports)
2. Spec Compliance — spawn `spec-reviewer` agent, verify every requirement
3. Code Quality — spawn `code-quality-reviewer` agent (only if spec passes)
4. Test Verification — run tests, check coverage, edge cases
5. Manual Review Summary — prepare diff + findings for user
6. Approval — **HARD GATE** — SHIP → Phase 8, NEEDS WORK → Phase 6, BLOCKED → Phase 4/5

**On SHIP + approve → show progress, continue to Phase 8.**

---

# Phase 8: Documentation

Read `.claude/skills/documentation/SKILL.md` and execute it fully — all 5 steps.

**Summary of steps:**
1. Context (silent) — read Phase 7 review + actual code
2. API Docs — document new/changed endpoints from CODE (source of truth)
3. Dev Notes — how it works, architecture decisions, gotchas
4. Changelog Entry — Keep a Changelog format
5. Approval + Document — **HARD GATE** — write to `.planning/active/08-documentation/`

**On approve → show progress, continue to Phase 9.**

---

# Phase 9: Deploy

Read `.claude/skills/deploy/SKILL.md` and execute it fully — all 6 steps.

**Summary of steps:**
1. Context (silent) — read all phase artifacts
2. Pre-deploy Checklist — tests, review verdict, secrets, migrations, docs
3. Migration Plan — if DB changes (expand-contract for breaking changes)
4. Merge/PR — create PR with summary or merge locally
5. Post-deploy Verification — health check, smoke tests, logs
6. Archive + Approval — **HARD GATE** — archive `.planning/active/` → `docs/features/`

**On approve → DONE. Pipeline complete.**

---

## Resuming

If interrupted, say "continue" or run `/workflow-full continue`.
Reads `.planning/STATE.md` → resumes from current phase.
