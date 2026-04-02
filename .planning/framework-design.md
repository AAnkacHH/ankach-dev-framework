# Ankach Dev Framework — Design Document

**Status:** IMPLEMENTED --- testing needed
**Date:** 2026-03-28
**Based on:** Superpowers, GSD, Everything Claude Code, Antigravity Awesome Skills

---

## Overview

Personal development workflow framework for Claude Code. 9-phase pipeline with manual gates at every step. Works for both greenfield (new projects) and brownfield (adding features to existing projects). Tech-agnostic — stack specifics live in CLAUDE.md, skills work with any stack.

## Core Principles

1. **Every phase = manual gate** — agent always waits for explicit `approve` / `reject`
2. **Every phase = artifacts** — concrete `.md` files in `.planning/active/`
3. **Brownfield + greenfield** — auto-detects project type, adapts research strategy
4. **Everything documented** — full decision history saved to `.md` files
5. **Two validation loops** — Phase 5 can loop back to 3/4, Phase 7 can loop back to 4/5/6
6. **Tech-agnostic skills** — stack specifics only in CLAUDE.md
7. **File-based inter-phase communication** — agents communicate via `.planning/` files
8. **Anti-rationalization** — every skill has excuse→reality tables to prevent skipping steps
9. **Macro/micro separation** — architecture (between systems) vs design (within system)

---

## Framework File Structure

```
.claude/
├── commands/                             # Slash command triggers (thin)
│   ├── analyze.md                        # /analyze → Phase 1
│   ├── architect.md                      # /architect → Phase 2
│   ├── design.md                         # /design → Phase 3
│   ├── decompose.md                      # /decompose → Phase 4
│   ├── validate.md                       # /validate → Phase 5
│   ├── build.md                          # /build → Phase 6
│   ├── review.md                         # /review → Phase 7
│   ├── document.md                       # /document → Phase 8
│   └── deploy.md                         # /deploy → Phase 9
│
├── skills/                               # Full workflow per phase
│   ├── analysis/SKILL.md                 # Phase 1: 6-step workflow
│   ├── architecture/SKILL.md             # Phase 2: macro architecture
│   ├── design/SKILL.md                   # Phase 3: micro design
│   ├── decomposition/SKILL.md            # Phase 4: EPIC → STORIES → TASKS
│   ├── validation/SKILL.md               # Phase 5: requirements validation
│   ├── implementation/SKILL.md           # Phase 6: parallel implementation
│   ├── review/SKILL.md                   # Phase 7: CR + tests
│   ├── documentation/SKILL.md            # Phase 8: docs
│   └── deploy/SKILL.md                   # Phase 9: deploy
│
└── agents/                               # Sub-agent definitions
    └── researcher.md                     # Research agent (greenfield + deep research)
    # Future: implementer.md, spec-reviewer.md, code-quality-reviewer.md, etc.
```

---

## 9-Phase Pipeline

```
                    ┌──────────────────────────────┐
                    │  Phase 1: Analysis           │ GATE
                    │  what + which approach        │
                    └──────────────┬───────────────┘
                                   │
                    ┌──────────────▼───────────────┐
                    │  Phase 2: Architecture       │ GATE
                    │  macro: servers, comms, tech  │
                    └──────────────┬───────────────┘
                                   │
                    ┌──────────────▼───────────────┐
                    │  Phase 3: Design             │ GATE
                    │  micro: entities, contracts   │
                    └──────────────┬───────────────┘
                                   │
                    ┌──────────────▼───────────────┐
                    │  Phase 4: Decomposition      │ GATE
                    │  EPIC→STORIES→TASKS→SUBTASKS  │
                    └──────────────┬───────────────┘
                                   │
                    ┌──────────────▼───────────────┐
              ┌────►│  Phase 5: Validation         │ GATE ──── reject ── back to 3/4
              │     │  requirements check           │
              │     └──────────────┬───────────────┘
              │                    │ approve
              │     ┌──────────────▼───────────────┐
              │     │  Phase 6: Implementation     │ GATE
              │     │  parallel agents              │
              │     └──────────────┬───────────────┘
              │                    │
              │     ┌──────────────▼───────────────┐
              └─────│  Phase 7: Review             │ GATE ──── reject ── back to 4/5/6
                    │  CR + tests                   │
                    └──────────────┬───────────────┘
                                   │ approve
                    ┌──────────────▼───────────────┐
                    │  Phase 8: Documentation      │ GATE
                    └──────────────┬───────────────┘
                                   │
                    ┌──────────────▼───────────────┐
                    │  Phase 9: Deploy             │ GATE
                    └──────────────────────────────┘
```

---

## Phase Definitions

### Phase 1: Analysis (DESIGNED)

**Purpose:** Understand WHAT we're building and choose an approach.
**Status:** Design complete — see `.planning/research/phase1-agent-proposal.md`

| Step | Name | Gate? |
|------|------|-------|
| 1 | Context (silent) | - |
| 2 | Research (brownfield: codebase / greenfield: external) | - |
| 3 | Questions (one at a time, max 7) | - |
| 4 | Understanding Lock | **HARD GATE** |
| 5 | Approaches (2-3 with trade-offs) | - |
| 6 | Approval + Document | **HARD GATE** |

**Output:** `.planning/active/01-analysis/`

```
01-analysis/
├── index.md                    # Main analysis + decision log
├── codebase-research.md        # Brownfield: existing code findings
└── external-research.md        # Greenfield/optional: external sources
```

---

### Phase 2: Architecture (DESIGNED)

**Purpose:** Macro decisions — BETWEEN systems. Servers, communication, technologies, integrations.
**Status:** Design complete — see `.planning/research/phase2-3-architecture-comparison.md`

**What it covers:**
- Server / runtime decisions (new service? existing?)
- Inter-system communication (REST, message bus, webhooks, gRPC)
- Technology choices with rationale (new libraries, frameworks)
- External integrations (which APIs, which protocols)
- Infrastructure (queues, cache, new databases)
- Security boundaries (auth flow, credential handling)

**When it's short:** Adding feature to existing project — most tech already decided, only new integrations.
**When it's full:** New project or new microservice — full stack selection.

| Step | Name | Gate? |
|------|------|-------|
| 1 | Context (silent) — read Phase 1 analysis | - |
| 2 | Gray Areas — find ambiguities in macro decisions | - |
| 3 | Questions — tech decisions: communication, integrations, infra | - |
| 4 | Architecture Decisions — ADR for each tech choice (Context → Options → Decision → Rationale → Consequences) | - |
| 5 | System Design Checklist — verify nothing missed (functional, non-functional, technical, operations) | - |
| 6 | Approval + Document | **HARD GATE** |

**Key patterns:**
- **Gray areas** (GSD) — find ambiguities, present for user decision
- **ADR format** (ECC + Antigravity) — documented rationale for every tech choice
- **"3 Questions" gate** (Antigravity) — before each pattern: what problem? simpler alternative? can we defer?
- **System Design Checklist** (ECC) — functional, non-functional, technical, operations

**Output:** `.planning/active/02-architecture/`

```
02-architecture/
├── index.md                    # Macro decisions summary + system design checklist
├── tech-decisions.md           # ADRs: each decision with rationale
├── integrations.md             # External services, protocols, data flow
└── infrastructure.md           # Queues, cache, new services (if applicable)
```

---

### Phase 3: Design (DESIGNED)

**Purpose:** Micro decisions — WITHIN the system. Domain structure, entities, API contracts.
**Status:** Design complete — see `.planning/research/phase2-3-architecture-comparison.md`

**What it covers:**
- Domain map: subdomains, bounded contexts, relationships, diagrams
- Data model: entities, fields, relations, migrations
- API contracts: endpoints, request/response DTOs, validation rules, status codes
- Requirements traceability: each requirement mapped to domain area

**When it's short:** Small feature — 1 subdomain affected, 1 entity, 1 endpoint.
**When it's full:** New module — multiple subdomains, multiple entities, multiple endpoints.

| Step | Name | Gate? |
|------|------|-------|
| 1 | Context (silent) — read Phase 2 architecture | - |
| 2 | Domain Map — subdomains, bounded contexts, relationships, new vs existing | - |
| 3 | Data Model — entities, fields, relations, migrations | - |
| 4 | API Contracts — endpoints, DTOs, validation, response format, status codes | - |
| 5 | Self-Review — coverage, consistency, no placeholders | - |
| 6 | Approval + Document | **HARD GATE** |

**Key patterns:**
- **Domain map** — subdomains with responsibilities, relationships, impact assessment (major/minor/none)
- **"No placeholders" rule** (Superpowers) — concrete fields, types, validation rules
- **API contract format** (Antigravity + ECC) — response envelope, status codes, pagination
- **Self-review** (Superpowers) — 3 checks: spec coverage, placeholder scan, consistency

**Output:** `.planning/active/03-design/`

```
03-design/
├── index.md                    # Summary + requirements traceability
├── domain-map.md               # Subdomains, bounded contexts, diagrams, impact
├── data-model.md               # Entities, fields, relations, migrations
└── api-contracts.md            # Endpoints, DTOs, validation, status codes
```

---

### Phase 4: Decomposition (DESIGNED)

**Purpose:** Break the design into executable work units: EPIC → STORIES → SUB-TASKS. Based on existing `docs/requirements/` structure with added story-level dependencies and wave plan.
**Status:** Design complete — see `.planning/research/phase4-decomposition-comparison.md`

**What it covers:**
- Epic definition with goal, scope, dependencies on other epics
- Stories with priority, estimate, spec reference, acceptance criteria
- Sub-tasks as checkboxes with exact class/file/action (designed for AI agent delegation)
- Story-level dependencies (`Depends on: Story 4.1`) for agent understanding
- Wave plan within epic for parallel story execution

**Enhancement over existing `docs/requirements/`:** Added `Depends on:` and `Wave:` per story. Format otherwise unchanged — it already works well.

| Step | Name | Gate? |
|------|------|-------|
| 1 | Context (silent) — read Phase 3 design (domain map, data model, API contracts) | - |
| 2 | Epic Definition — goal, scope, acceptance criteria, dependencies on other epics | - |
| 3 | Story Decomposition — stories with dependencies between stories, priorities, estimates | - |
| 4 | Sub-task Breakdown — per story, exact files/classes/actions for AI delegation | - |
| 5 | Wave Plan — parallel groups of stories within the epic | - |
| 6 | Approval + Document | **HARD GATE** |

**Key patterns:**
- **Existing format preserved** — Epic (index.md) + Stories with Sub-tasks (tasks.md)
- **Story dependencies** added — `Depends on: Story X.Y` enables agent understanding of execution order
- **Wave plan per epic** — which stories can run in parallel (not just epic-level waves)
- **Sub-tasks designed for AI delegation** — exact file, exact class, exact properties

**Story format (enhanced):**
```markdown
### Story 4.2: Carrier Catalog Endpoints
**Priority:** P0 | **Estimate:** 4h | **Wave:** B
**Spec:** spec/03_api_specification.md (Section 4.2)
**Depends on:** Story 4.1 (API Infrastructure)

> Description...

**Sub-tasks:**
- [ ] 4.2.1 Create `CarriersController` in `CarrierPlatform.Api/Controllers/`...
- [ ] 4.2.2 Implement `GET /api/v1/carriers`...

**Acceptance Criteria:**
- [ ] `GET /api/v1/carriers` returns paginated list...
```

**Output:** `.planning/active/04-decomposition/`

```
04-decomposition/
├── index.md                    # Epic overview + dependency diagram
├── epic.md                     # Epic definition: goal, scope, AC
├── stories/
│   ├── story-01-{slug}.md      # Story + sub-tasks + acceptance criteria
│   ├── story-02-{slug}.md      #   each with Depends on: + Wave:
│   └── ...
└── wave-plan.md                # Wave groups within the epic
```

---

### Phase 5: Validation (DESIGNED)

**Purpose:** Verify architecture quality and requirements completeness before implementation. Can loop back to Phase 3/4. Max 2 revision iterations → escalate to user.
**Status:** Design complete — see `.planning/research/phase5-validation-comparison.md`

**What it covers:**
- Architecture review: deprecated tech, anti-patterns, security, performance, over/under-engineering, framework misuse, compatibility
- Requirements coverage: every Phase 3 requirement → Phase 4 story
- Story completeness: AC, dependencies, wave, sub-tasks
- Design alignment: decomposition matches domain map, data model, API contracts
- Scope sanity + no placeholders

| Step | Name | Gate? |
|------|------|-------|
| 1 | Context (silent) — read Phase 2 + 3 + 4 | - |
| 2 | Architecture Review — check tech decisions for deprecated/EOL, anti-patterns, security, performance, over/under-engineering, framework misuse, compatibility | - |
| 3 | Requirements Coverage — every requirement from Phase 3 maps to a story in Phase 4 | - |
| 4 | Story Completeness — each story has AC, depends on, wave, sub-tasks | - |
| 5 | Alignment + Scope + Placeholders — decomposition matches design, no scope creep, no TBD | - |
| 6 | Validation Report + Approval | **HARD GATE** — approve → Phase 6, reject → back to 3/4 |

**Architecture Review checks:**

| Category | What | Example |
|----------|------|---------|
| Deprecated/EOL | Technologies no longer supported | Newtonsoft.Json → System.Text.Json |
| Anti-patterns | Typical architecture mistakes | God service, tight coupling, N+1, anemic models |
| Security | Vulnerabilities in design | Credentials in DTO, missing rate limiting |
| Performance | Obvious bottlenecks | Sync where async needed, missing indexes |
| Over-engineering | Unnecessary complexity | CQRS for CRUD, microservices for monolith task |
| Under-engineering | Missing necessary things | No error handling, no retry policy for carrier API |
| Framework misuse | Incorrect framework usage | HasDefaultValue(enum) in EF, DbContext in controller |
| Compatibility | Versions, breaking changes | Library doesn't support .NET 10 |

**Findings severity:** BLOCKER (must fix) / WARNING (should fix) / INFO (consider)
**On FAIL:** loop back to Phase 3 or 4 to fix. Max 2 iterations → escalate to user.
**Research agent:** spawned for version/compatibility checks via context7 + web search.

**Output:** `.planning/active/05-validation/`

```
05-validation/
├── index.md                    # PASS / FAIL + summary
├── architecture-review.md      # Tech review: findings with severity
├── checklist.md                # 4 checks (coverage, completeness, alignment, scope)
├── gaps.md                     # Specific issues found (if any)
└── revision-log.md             # If loop — what was fixed and why
```

---

### Phase 6: Implementation (DESIGNED)

**Purpose:** Execute code via parallel agents following the wave plan from Phase 4. Fresh context per story, atomic commits, deviation rules for autonomy control.
**Status:** Design complete — see `.planning/research/phase6-implementation-comparison.md`

| Step | Name | Gate? |
|------|------|-------|
| 1 | Context (silent) — read Phase 4 wave-plan + Phase 5 validation | - |
| 2 | Setup — git branch, verify baseline tests pass | - |
| 3 | Execute Wave — dispatch agents for current wave stories | - |
| 4 | Wave Report — progress, concerns, issues | **GATE per wave** — approve next wave |
| 5 | Implementation Summary — all waves done, full report | - |
| 6 | Approval | **HARD GATE** → Phase 7 |

**Execution model:**
- Wave-based parallel: stories within same wave run in parallel agents
- Fresh context per story (no context pollution)
- Agent receives: full story text + CLAUDE.md rules + Phase 3 design files
- Atomic git commit per sub-task
- User approves after each wave before next wave starts

**Deviation rules (from GSD):**

| Rule | Trigger | Action |
|------|---------|--------|
| Rule 1 (Bug) | Broken behavior, errors | Auto-fix, max 3 attempts |
| Rule 2 (Missing Critical) | No validation, auth, error handling | Auto-add, max 3 attempts |
| Rule 3 (Blocking) | Missing deps, config issues | Auto-resolve, max 3 attempts |
| Rule 4 (Architectural) | New tables, major restructuring | **STOP — user decides** |

**Agent status codes:** DONE / DONE_WITH_CONCERNS / BLOCKED / NEEDS_CONTEXT

**Guards:**
- Analysis paralysis: 5+ reads without writes = STOP, report blocker
- Scope boundary: only fix issues caused by current task, pre-existing → issues.md

**Output:** `.planning/active/06-implementation/`

```
06-implementation/
├── index.md                    # Overall progress by waves
├── agents/
│   ├── wave-A-story-01.md      # Agent report: what done, files, concerns
│   ├── wave-B-story-02.md
│   ├── wave-B-story-03.md
│   └── ...
└── issues.md                   # Blockers, Rule 4 decisions, deviations
```

---

### Phase 7: Review (DESIGNED)

**Purpose:** Three-stage auto review (spec compliance → code quality → tests) + manual review gate. Can loop back to Phase 6 for fixes, or Phase 4/5 for redesign.
**Status:** Design complete — see `.planning/research/phase7-review-comparison.md`

| Step | Name | Gate? |
|------|------|-------|
| 1 | Context — read Phase 6 implementation reports | - |
| 2 | Spec Compliance — auto: code matches Phase 3 design? Read CODE, not reports | - |
| 3 | Code Quality — auto: SOLID, security, performance, naming, conventions | - |
| 4 | Test Verification — run tests, check coverage, verify edge cases | - |
| 5 | Manual Review Summary — diff overview + auto-review findings for human | - |
| 6 | Approval | **HARD GATE** — SHIP → Phase 8, NEEDS WORK → Phase 6, BLOCKED → Phase 4/5 |

**Staged order (fail-fast):**
- Spec compliance fails → back to Phase 6 (fix implementation)
- Code quality fails → back to Phase 6 (fix quality)
- Tests fail → back to Phase 6 (fix tests)
- All auto-checks pass → manual review summary → user decides

**Key patterns:**
- **"Do not trust the report"** (Superpowers) — reviewer reads actual code, not agent reports
- **Staged gates** — spec compliance BEFORE quality (don't review quality of wrong code)
- **Pushback encouraged** — implementer can argue against feedback if YAGNI or technically wrong
- **Iron Law** — no completion claims without fresh `dotnet test` output as evidence

**Spec compliance checks:** every Phase 3 requirement implemented, no scope creep, no misunderstandings
**Code quality checks:** SOLID, security (SQL injection, XSS, credentials), performance (N+1, async), naming, file size (<800), nesting (<4), error handling, CLAUDE.md conventions
**Test verification:** all pass, coverage report, edge cases, tests test logic not mocks

**Verdict:** `SHIP` / `NEEDS WORK` / `BLOCKED`
**Max 2 iterations** fix → re-review → approve or escalate to user

**Output:** `.planning/active/07-review/`

```
07-review/
├── index.md                    # Verdict: SHIP / NEEDS WORK / BLOCKED
├── spec-compliance.md          # Auto: requirement-by-requirement check
├── code-quality.md             # Auto: findings with severity (CRITICAL/HIGH/MEDIUM/LOW)
├── test-results.md             # Test output + coverage
└── revision-log.md             # If loop — what was fixed
```

---

### Phase 8: Documentation (DESIGNED)

**Purpose:** Generate developer documentation, API docs, and changelog entry. Source of truth = code, not manual descriptions.
**Status:** Design complete — see `.planning/research/phase8-9-docs-deploy-comparison.md`

| Step | Name | Gate? |
|------|------|-------|
| 1 | Context — read Phase 7 review results + implemented code | - |
| 2 | API Docs — document new/changed endpoints (request, response, errors, examples) | - |
| 3 | Dev Notes — how it works internally, architecture decisions, gotchas | - |
| 4 | Changelog Entry — what changed, why, migration notes if any | - |
| 5 | Approval + Document | **HARD GATE** |

**Key patterns:**
- **"Generate from source of truth"** (ECC) — read actual code, not Phase 3 design docs
- **Changelog** — Keep a Changelog format: Added/Changed/Fixed/Removed
- **Dev notes** — for future developers/agents: how components interact, non-obvious decisions

**Output:** `.planning/active/08-documentation/`

```
08-documentation/
├── index.md                    # Summary of what was documented
├── api-docs.md                 # New/changed endpoints documentation
├── dev-notes.md                # How it works, architecture decisions, gotchas
└── changelog-entry.md          # Entry for CHANGELOG.md
```

---

### Phase 9: Deploy (DESIGNED)

**Purpose:** Pre-deploy checklist, database migrations, merge/PR, post-deploy verification.
**Status:** Design complete — see `.planning/research/phase8-9-docs-deploy-comparison.md`

| Step | Name | Gate? |
|------|------|-------|
| 1 | Context — read all previous phase artifacts | - |
| 2 | Pre-deploy Checklist — tests pass, no secrets, migrations ready, docs updated | - |
| 3 | Migration Plan — DB migrations if any (expand-contract for breaking changes) | - |
| 4 | Merge/PR — create PR with summary from planning artifacts, or merge locally | - |
| 5 | Post-deploy Verification — smoke tests, health checks, verify key flows | - |
| 6 | Archive + Approval | **HARD GATE** — archive .planning/active/ → docs/features/ |

**Pre-deploy checklist:**
- [ ] All tests pass (`dotnet test`)
- [ ] Phase 7 review verdict = SHIP
- [ ] No CRITICAL issues in review
- [ ] No hardcoded secrets in code
- [ ] Database migrations tested
- [ ] Documentation updated (Phase 8)
- [ ] CHANGELOG entry written
- [ ] Environment variables verified

**Merge strategy:** configurable in CLAUDE.md (squash recommended for features, merge for large epics)

**Post-deploy verification:**
- Health check endpoints respond
- Key user flows work (smoke test)
- No new errors in logs (first 5 minutes)

**Archive:** on completion, copy `.planning/active/` → `docs/features/YYYY-MM-DD-{slug}/`

**Output:** `.planning/active/09-deploy/`

```
09-deploy/
├── index.md                    # Deploy status + summary
├── pre-deploy-checklist.md     # All checks with pass/fail
├── migration-plan.md           # DB migrations (if any)
└── post-deploy-verification.md # Smoke test results
```

---

## Agents

| Agent | Phase | Tools | Model | Purpose |
|-------|-------|-------|-------|---------|
| `researcher` | 1, 2 | Read, Write, Grep, Glob, Bash, WebSearch, WebFetch | sonnet | Codebase + external research |
| `implementer` | 6 | TBD | sonnet | Implement a single task |
| `test-writer` | 6 | TBD | sonnet | Write tests for implementation |
| `spec-reviewer` | 7 | TBD (READ-ONLY) | sonnet | Check spec compliance |
| `code-quality-reviewer` | 7 | TBD (READ-ONLY) | opus | Check code quality |

---

## STATE.md Template

```markdown
# Feature State

| Field          | Value                    |
|----------------|--------------------------|
| Feature        | {Feature Name}           |
| Slug           | {feature-slug}           |
| Started        | {YYYY-MM-DD}             |
| Current Phase  | {N} — {Phase Name}       |
| Status         | AWAITING_APPROVAL / IN_PROGRESS / APPROVED / REJECTED |
| Branch         | feature/{slug}           |
| Type           | brownfield / greenfield  |

## Phase Log

| Phase | Status    | Approved   | Attempts | Notes                          |
|-------|-----------|------------|----------|--------------------------------|
| 1     | completed | YYYY-MM-DD | 1        |                                |
| 2     | pending   | —          | 0        |                                |

## Decision Log

- [Phase 1] Chose approach B: Wolverine batch processing
- [Phase 2] Added Redis for rate limiting
- [Phase 3] BulkShipmentRequest entity with 5 fields
```

---

## What We Took from Each Framework

### From Superpowers
- Anti-rationalization tables (excuse → reality) in every skill
- `<HARD-GATE>` XML tags for enforcement
- Description = trigger only, NOT summary
- One question per message rule
- Spec self-review (4 checks: placeholders, contradictions, scope, ambiguity)

### From GSD
- `.planning/` as shared state between phases
- Brownfield/greenfield auto-detection
- Parallel research agents for deep domain research
- Deep questioning: "dream extraction, not requirements gathering"
- Gray areas identification (find ambiguities, present for user decision)
- CONTEXT.md as handoff between "what" and "how" phases
- Atomic commits per phase artifact
- Wave-based parallel execution (Phase 6)
- Goal-backward verification (Phase 7)

### From Everything Claude Code
- Tool whitelist per agent (READ-ONLY for reviewers)
- Model routing (opus for quality review, sonnet for implementation)
- Worked examples showing expected output quality
- Structured verdicts: SHIP / NEEDS WORK / BLOCKED
- Handoff documents between agents/phases
- Context switching (research mode for analysis)
- Architect (what/why) vs Planner (how/when) agent separation
- System Design Checklist (functional, non-functional, technical, operations)
- ADR in Nygard format with lifecycle management

### From Antigravity
- Understanding Lock (summary → confirm → then design)
- Decision Log (mandatory, grows across phases)
- Risk-based escalation (low/moderate/high)
- Exit criteria as hard stops (ALL must be true)
- State machine patterns
- Quality gates for skills themselves
- "3 Questions" gate before any pattern (problem? simpler? defer?)
- Pattern Selection decision trees (data access, business rules, scaling)
- ADR templates (5 formats from lightweight to RFC)

---

## Research Archive

- `.planning/research/phase1-analysis-comparison.md` — comparison of 4 frameworks for Phase 1
- `.planning/research/phase1-agent-proposal.md` — Phase 1 agent design (v2, 6-step)
- `.planning/research/phase2-3-architecture-comparison.md` — comparison of 4 frameworks for Phase 2-3
- `.planning/research/phase4-decomposition-comparison.md` — comparison of 4 frameworks for Phase 4 vs existing structure
- `.planning/research/phase5-validation-comparison.md` — comparison of 4 frameworks for Phase 5
- `.planning/research/phase6-implementation-comparison.md` — comparison of 4 frameworks for Phase 6
- `.planning/research/phase7-review-comparison.md` — comparison of 4 frameworks for Phase 7
- `.planning/research/phase8-9-docs-deploy-comparison.md` — comparison of 4 frameworks for Phase 8-9

---

## Implementation Progress

### Phase 1: Analysis
- [x] Research frameworks (4 parallel agents)
- [x] Deep-dive analysis phase comparison
- [x] Design 6-step workflow
- [x] Implement SKILL.md
- [x] Implement /analyze command
- [x] Implement researcher agent
- [ ] Test on real feature

### Phase 2: Architecture
- [x] Research how frameworks handle macro architecture
- [x] Design 6-step workflow (gray areas, ADRs, "3 Questions" gate, checklist)
- [x] Implement SKILL.md
- [x] Implement /architect command
- [ ] Test on real feature

### Phase 3: Design
- [x] Research how frameworks handle micro design
- [x] Design 6-step workflow (domain map, data model, API contracts, self-review)
- [x] Implement SKILL.md
- [x] Implement /design command
- [ ] Test on real feature

### Phase 4: Decomposition
- [x] Research how frameworks handle task decomposition
- [x] Compare with existing docs/requirements/ structure
- [x] Design — keep existing format + add story dependencies + wave plan
- [x] Implement SKILL.md
- [x] Implement /decompose command
- [ ] Test on real feature

### Phase 5: Validation
- [x] Research how frameworks handle pre-implementation validation
- [x] Design — 6-step workflow with architecture review + 4 quality checks
- [x] Implement SKILL.md
- [x] Implement /validate command
- [ ] Test on real feature

### Phase 6: Implementation
- [x] Research how frameworks handle parallel implementation
- [x] Design — wave-based parallel, deviation rules, status codes, guards
- [x] Implement SKILL.md
- [x] Implement /build command
- [x] Implement implementer agent
- [ ] Test on real feature

### Phase 7: Review
- [x] Research how frameworks handle code review
- [x] Design — 3-stage auto review (spec → quality → tests) + manual gate
- [x] Implement SKILL.md
- [x] Implement /review command
- [x] Implement spec-reviewer + code-quality-reviewer agents
- [ ] Test on real feature

### Phase 8: Documentation
- [x] Research how frameworks handle documentation generation
- [x] Design — 5-step workflow (API docs, dev notes, changelog)
- [x] Implement SKILL.md
- [x] Implement /document command
- [ ] Test on real feature

### Phase 9: Deploy
- [x] Research how frameworks handle deployment
- [x] Design — 6-step workflow (checklist, migrations, merge/PR, post-deploy, archive)
- [x] Implement SKILL.md
- [x] Implement /deploy command
- [ ] Test on real feature

### Infrastructure
- [x] Directory structure created
- [x] CLAUDE.md framework section
- [ ] Hooks for enforcement
- [ ] Archive workflow (active/ → docs/features/)
