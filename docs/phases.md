# 9 Phases

```
 ┌─────────────────────────┐
 │  1. Analysis             │  Understand the task, research, propose approaches
 └────────────┬────────────┘
              ▼
 ┌─────────────────────────┐
 │  2. Architecture         │  Macro: servers, communication, technologies
 └────────────┬────────────┘
              ▼
 ┌─────────────────────────┐
 │  3. Design               │  Micro: domain map, entities, API contracts
 └────────────┬────────────┘
              ▼
 ┌─────────────────────────┐
 │  4. Decomposition        │  EPIC → STORIES → TASKS with wave plan
 └────────────┬────────────┘
              ▼
 ┌─────────────────────────┐
 │  5. Validation           │  Architecture review + requirements coverage
 └────────────┬────────────┘  ↺ loop to 3/4 on FAIL
              ▼
 ┌─────────────────────────┐
 │  6. Implementation       │  Parallel agents, wave-by-wave execution
 └────────────┬────────────┘
              ▼
 ┌─────────────────────────┐
 │  7. Review               │  Spec compliance → Code quality → Tests
 └────────────┬────────────┘  ↺ loop to 4/5/6 on NEEDS WORK
              ▼
 ┌─────────────────────────┐
 │  8. Documentation        │  API docs, dev notes, changelog
 └────────────┬────────────┘
              ▼
 ┌─────────────────────────┐
 │  9. Deploy               │  Checklist, merge/PR, verify, archive
 └─────────────────────────┘
```

---

## Phase 1: Analysis (`/analyze`)

**Purpose:** Understand WHAT we're building and choose an approach.

**6 Steps:**
1. **Context** (silent) — read CLAUDE.md, AGENTS.md, .context/, detect brownfield/greenfield
2. **Research** — brownfield: codebase patterns; greenfield: external research via `researcher` agent
3. **Questions** — one at a time, max 7, multiple choice preferred
4. **Understanding Lock** — present summary, get explicit confirmation (HARD GATE)
5. **Approaches** — 2-3 with trade-offs, recommend one
6. **Approval + Document** — write artifacts, get approval (HARD GATE)

**Output:** `.planning/active/01-analysis/`
- `index.md` — main analysis + decision log
- `codebase-research.md` — existing code findings (brownfield)
- `external-research.md` — external sources (greenfield)

---

## Phase 2: Architecture (`/architect`)

**Purpose:** Macro decisions — BETWEEN systems.

**6 Steps:**
1. **Context** (silent) — read Phase 1 analysis
2. **Gray Areas** — find ambiguities in macro decisions
3. **Questions** — tech decisions, "3 Questions" gate per pattern
4. **Architecture Decisions** — ADR for each choice (Context → Options → Decision → Rationale)
5. **System Design Checklist** — functional, non-functional, technical, operations
6. **Approval + Document** — HARD GATE

**Output:** `.planning/active/02-architecture/`
- `index.md` — macro decisions summary + checklist
- `tech-decisions.md` — ADRs
- `integrations.md` — external services, protocols
- `infrastructure.md` — queues, cache, new services

---

## Phase 3: Design (`/design`)

**Purpose:** Micro decisions — WITHIN the system.

**6 Steps:**
1. **Context** (silent) — read Phase 2 architecture
2. **Domain Map** — subdomains, bounded contexts, relationships, impact assessment
3. **Data Model** — entities, fields, relations, migrations
4. **API Contracts** — endpoints, DTOs, validation, status codes
5. **Self-Review** — coverage, consistency, no placeholders
6. **Approval + Document** — HARD GATE

**Output:** `.planning/active/03-design/`
- `index.md` — summary + requirements traceability
- `domain-map.md` — subdomains, relationships, impact
- `data-model.md` — entities, fields, relations
- `api-contracts.md` — endpoints, DTOs, validation

---

## Phase 4: Decomposition (`/decompose`)

**Purpose:** Break design into executable work units.

**6 Steps:**
1. **Context** (silent) — read Phase 3 design
2. **Epic Definition** — goal, scope, acceptance criteria
3. **Story Decomposition** — stories with `Depends on:` + `Wave:` + sub-tasks
4. **Sub-task Breakdown** — exact files, classes, actions for AI delegation
5. **Wave Plan** — parallel groups within stories
6. **Approval + Document** — HARD GATE

**Output:** `.planning/active/04-decomposition/`
- `index.md` — epic overview + dependency diagram
- `epic.md` — epic definition
- `stories/story-{NN}-{slug}.md` — stories with sub-tasks
- `wave-plan.md` — parallel execution waves

---

## Phase 5: Validation (`/validate`)

**Purpose:** Quality gate before coding. Catches design issues that would be expensive to fix.

**6 Steps:**
1. **Context** (silent) — read Phase 2 + 3 + 4
2. **Architecture Review** — deprecated tech, anti-patterns, security, performance, framework misuse
3. **Requirements Coverage** — every Phase 3 requirement → Phase 4 story
4. **Story Completeness + Alignment + Scope + Placeholders**
5. **Validation Report** — PASS/FAIL with issues
6. **Approval** — HARD GATE. FAIL → back to Phase 3/4 (max 2 iterations)

**Output:** `.planning/active/05-validation/`
- `index.md` — PASS/FAIL + summary
- `architecture-review.md` — findings with severity
- `checklist.md` — coverage + completeness checks
- `gaps.md` — specific issues
- `revision-log.md` — if loop, what changed

---

## Phase 6: Implementation (`/build`)

**Purpose:** Execute code via parallel agents.

**6 Steps:**
1. **Context** (silent) — read Phase 4 wave-plan + Phase 5 validation
2. **Setup** — git branch, verify baseline tests pass
3. **Execute Waves** — dispatch `implementer` agents per wave
4. **Wave Report** — progress after each wave, user approves next (GATE per wave)
5. **Implementation Summary** — all waves done
6. **Approval** — HARD GATE

**Output:** `.planning/active/06-implementation/`
- `index.md` — progress by waves
- `agents/wave-{X}-story-{NN}.md` — per-agent reports
- `issues.md` — blockers, deviations

---

## Phase 7: Review (`/review`)

**Purpose:** Three-stage code review + manual gate.

**6 Steps:**
1. **Context** — read Phase 6 reports + actual code
2. **Spec Compliance** — spawn `spec-reviewer` agent (read CODE, not reports)
3. **Code Quality** — spawn `code-quality-reviewer` agent (only if spec passes)
4. **Test Verification** — run tests, check coverage
5. **Manual Review Summary** — diff + findings for user
6. **Approval** — HARD GATE. SHIP → Phase 8, NEEDS WORK → Phase 6, BLOCKED → Phase 4/5

**Output:** `.planning/active/07-review/`
- `index.md` — verdict: SHIP / NEEDS WORK / BLOCKED
- `spec-compliance.md` — requirement-by-requirement
- `code-quality.md` — findings with severity
- `test-results.md` — test output + coverage
- `revision-log.md` — if fixes applied

---

## Phase 8: Documentation (`/document`)

**Purpose:** Generate docs from code (source of truth).

**5 Steps:**
1. **Context** (silent) — read Phase 7 review + actual code
2. **API Docs** — new/changed endpoints
3. **Dev Notes** — how it works, gotchas
4. **Changelog Entry** — Keep a Changelog format
5. **Approval + Document** — HARD GATE

**Output:** `.planning/active/08-documentation/`
- `index.md` — what was documented
- `api-docs.md` — endpoint documentation
- `dev-notes.md` — developer guide
- `changelog-entry.md` — changelog entry

---

## Phase 9: Deploy (`/deploy`)

**Purpose:** Ship it.

**6 Steps:**
1. **Context** (silent) — read all phase artifacts
2. **Pre-deploy Checklist** — tests, review, secrets, migrations, docs
3. **Migration Plan** — if DB changes
4. **Merge/PR** — create PR or merge locally
5. **Post-deploy Verification** — health check, smoke tests
6. **Archive + Approval** — HARD GATE, archive to `docs/features/`

**Output:** `.planning/active/09-deploy/`
- `index.md` — deploy status
- `pre-deploy-checklist.md` — all checks
- `migration-plan.md` — DB migrations
- `post-deploy-verification.md` — smoke test results
