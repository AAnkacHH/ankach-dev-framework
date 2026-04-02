# Phase 2-3: Architecture & Design — Comparison of 4 Frameworks

**Date:** 2026-03-28

---

## How Each Framework Handles Macro vs Micro Architecture

### Superpowers — No Explicit Separation

Two documents, implicit split:
- **Spec** (macro) → `docs/superpowers/specs/` — approach selection, tech choices, components, data flow
- **Plan** (micro) → `docs/superpowers/plans/` — exact file paths:line numbers, complete code blocks, TDD steps

No separate architecture phase. Brainstorming produces spec, writing-plans produces plan.
Spec reviewer + plan reviewer as subagents with defined checklists.

**Key pattern:** "No placeholders rule" — every plan task has actual code, actual commands, actual expected output.

---

### GSD — 3 Layers + discuss/plan Separation

```
Layer 1: Project-level (macro, one-time)
  PROJECT.md, REQUIREMENTS.md, ROADMAP.md, codebase maps

Layer 2: Phase-level decisions (discuss-phase)
  CONTEXT.md — locked decisions (D-01, D-02...), canonical references, deferred ideas
  "WHAT do we want?" — user is visionary, Claude is builder

Layer 3: Phase-level plans (plan-phase)
  PLAN.md files — XML task structures, dependency graphs, wave assignments
  "HOW do we build it?" — orchestrator coordinating planner/checker agents
```

**Key insight:** discuss-phase identifies "gray areas" (ambiguities with multiple valid paths), presents them for user decision. Never asks about implementation details — those are planning concerns.

**Agents:**
- `gsd-phase-researcher` — external research for a specific phase
- `gsd-planner` — transforms decisions into executable PLAN.md files
- `gsd-plan-checker` — validates plans across 10 dimensions
- `gsd-assumptions-analyzer` — surfaces hidden assumptions with evidence

**Gates:**
- CONTEXT.md existence gate
- Scope guardrail (no capabilities outside phase boundary)
- UI design contract gate
- Plan verification gate (10 dimensions, max 3 revision iterations)
- Requirements coverage gate (every REQ-ID in at least one plan)

---

### ECC — Architect vs Planner Agents

Two separate agents with clear roles:

| | Architect | Planner |
|---|---|---|
| **Focus** | What and why | How and when |
| **Level** | System | Implementation |
| **Tools** | Read + Shell | Read only |
| **Model** | Opus | Opus |
| **Output** | ADRs, design docs, checklists | Step-by-step plans, file paths, risks |
| **Process** | State Analysis → Requirements → Design Proposal → Trade-offs | Requirements → Architecture Review → Steps → Implementation Order |

**Orchestration chains:**
- Feature: planner → tdd-guide → code-reviewer → security-reviewer
- Refactor: **architect** → code-reviewer → tdd-guide
- Security: security-reviewer → code-reviewer → **architect**

**ADRs:** Nygard format in `docs/adr/NNNN-title.md` with README index.
**Blueprint skill:** Multi-session plans with cold-start execution briefs per step.
**API design skill:** Response envelope, pagination, versioning, pre-ship checklist.
**System Design Checklist:** functional, non-functional, technical, operations.

---

### Antigravity — Rich Skill Library with Decision Trees

**Architecture skill** with modular sub-files:
- `context-discovery.md` — question hierarchy: Scale → Team → Timeline → Domain → Constraints
- `pattern-selection.md` — decision trees for data access, business rules, scaling, real-time
- `trade-off-analysis.md` — ADR templates (5 formats from lightweight to RFC)
- `examples.md` — worked examples: MVP, SaaS, Enterprise
- `patterns-reference.md` — quick lookup tables

**"3 Questions" gate before any pattern:**
1. What SPECIFIC problem does this pattern solve?
2. Is there a simpler solution?
3. Can we add this LATER when needed?

**Project Classification Matrix:**

|              | MVP        | SaaS       | Enterprise |
|--------------|------------|------------|------------|
| Scale        | <1K        | 1K-100K    | 100K+      |
| Team         | Solo       | 2-10       | 10+        |
| Architecture | Simple     | Modular    | Distributed|

**API patterns skill:** REST vs GraphQL vs tRPC decision tree, response formats, versioning strategies.
**Database design skill:** DB selection tree, schema design, indexing, migrations.
**DDD skill:** Viability check (2 of 4 criteria must be true), routing to sub-skills.
**Backend dev guidelines:** Layered architecture doctrine, BFRI scoring formula.

---

## Synthesis: What to Take for Ankach Framework

### For Phase 2: Architecture (macro)

| Pattern | Source | Why |
|---------|--------|-----|
| Context Discovery questions (scale, team, timeline, constraints) | Antigravity | Structured way to gather macro requirements |
| "Gray areas" identification | GSD | Find ambiguities, present for user decision |
| ADR format for tech decisions | ECC + Antigravity | Documented rationale for every tech choice |
| Pattern Selection decision trees | Antigravity | Systematic architecture choice, not gut feeling |
| "3 Questions" gate | Antigravity | Prevents over-engineering |
| System Design Checklist | ECC | Ensures nothing missed |
| Architect = read + shell (can inspect systems) | ECC | Can check running infra, not just read code |

### For Phase 3: Design (micro)

| Pattern | Source | Why |
|---------|--------|-----|
| Exact file mapping with line numbers | Superpowers | Precise scope for implementers |
| "No placeholders" rule | Superpowers | Forces concrete design, not hand-waving |
| Plan self-review (spec coverage, placeholder scan, type consistency) | Superpowers | Quality gate before user review |
| API contract design (response envelope, status codes, pagination) | Antigravity + ECC | Consistent API patterns |
| Data model design (entities, relations, indexing strategy) | Antigravity | Systematic schema decisions |
| Phased implementation (MVP → Core → Edge Cases → Optimization) | ECC | Each phase independently deliverable |
| Requirements coverage (every REQ-ID mapped to a file) | GSD | Nothing falls through cracks |
| Plan-checker validation (10 dimensions) | GSD | Automated quality check |
