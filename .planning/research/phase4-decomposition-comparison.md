# Phase 4: Decomposition — Comparison of 4 Frameworks vs Existing Structure

**Date:** 2026-03-29

---

## How Each Framework Decomposes Work

| | **Superpowers** | **GSD** | **ECC** | **Antigravity** | **Your Existing** |
|---|---|---|---|---|---|
| **Hierarchy** | Spec → Plan → Tasks → Steps | Milestone → Phase → Plan → Task | Feature → Phases → Steps | Issue → Plan → Tasks → Steps | Epic → Stories → Sub-tasks |
| **Top unit** | Feature spec | Phase (from roadmap) | Feature (one-line objective) | Issue (with DoD gate) | Epic |
| **Work unit** | Task (component) | Plan (2-3 tasks) | Step (one PR) | Task (TDD cycle) | Story |
| **Atomic unit** | Step (2-5 min) | Task (15-60 min) | Step (PR-sized) | Step (2-5 min) | Sub-task (checkbox) |
| **Parallelism** | None (sequential) | Wave-based at plan level | DAG with parallel detection | None (sequential) | Wave-based at epic level |
| **Dependencies** | Implicit (sequential) | `depends_on` in PLAN.md frontmatter | DAG edges | Issue `Dependencies` field | Mermaid graph between epics |
| **Verification** | TDD per step + two-stage review | `<verify>` + `<done>` per task | Exit criteria + verification commands | TDD + DoD evidence | Acceptance criteria per story |
| **File mapping** | Exact paths with line ranges | `<files>` in XML task | File paths in context brief | Exact paths per task | Mentioned in sub-task text |
| **Review** | Spec compliance → Code quality (per task) | Plan-checker (10 dimensions) | Adversarial review gate | Spec → Quality (per task) | Code review docs separate |

---

## Your Existing Structure (docs/requirements/)

```
9 Epics, 71 Stories, ~960 Sub-tasks
6 Waves for parallel execution
Dependency graph between epics (mermaid)
```

### What You Already Have (and It's GOOD)

| Pattern | Your Implementation | Status |
|---------|--------------------|--------|
| Epic → Stories → Sub-tasks hierarchy | 9 epics, well-structured | Excellent |
| Wave-based parallel execution | 6 waves at epic level | Excellent |
| Dependency graph | Mermaid diagrams, clear deps | Excellent |
| Sub-tasks for AI delegation | ~960 tasks with clear scope | Excellent |
| Spec references per story | `spec/03_api_specification.md (Section 4.1)` | Excellent |
| Acceptance criteria per story | Checkbox lists, testable | Excellent |
| Priority + estimates per story | P0/P1/P2 + hour estimates | Good |
| Deferred items list | Post-MVP table with priority | Good |

### Example of Your Existing Sub-task (Story 4.1):
```markdown
- [ ] 4.1.1 Create `ApiResponse<T>` generic wrapper class in
      `CarrierPlatform.Core/Models/Api/` with properties: `bool Success`,
      `T? Data`, `ApiMeta? Meta`, `ApiError? Error`. Include static factory
      methods `ApiResponse<T>.Ok(T data, ApiMeta? meta)` and
      `ApiResponse<T>.Fail(ApiError error, ApiMeta? meta)`.
```

This is already very detailed — exact class name, exact namespace, exact properties, exact methods. Better than most framework examples.

---

## What Could Be Improved (from frameworks)

### 1. No Task-Level Dependencies (only Epic-Level)

**Current:** Dependencies exist between epics (Epic 04 needs Epic 02 + 03). But within an epic, stories and sub-tasks have no explicit dependency markers.

**GSD does:** `depends_on: ["01-01"]` in each plan's frontmatter. Tasks within a plan are sequential, but plans can run in parallel waves.

**Recommendation:** Add wave groups WITHIN each epic for stories, not just between epics.

### 2. No Formal Verification Command Per Sub-task

**Current:** Acceptance criteria per story, but no explicit "run this command to verify" per sub-task.

**GSD does:**
```xml
<task>
  <verify>dotnet test --filter "ApiResponse"</verify>
  <done>ApiResponse<T> returns correct envelope for success and error cases</done>
</task>
```

**Superpowers does:**
```markdown
- [ ] Step 4: Run test to verify it passes
Run: `dotnet test tests/.../ApiResponseTests.cs -v`
Expected: PASS
```

**Recommendation:** Add `Verify:` field to sub-tasks where applicable.

### 3. No Two-Stage Review Per Story

**Current:** Code reviews are separate documents (`code-review-*.md`). Not embedded in the decomposition.

**Superpowers/Antigravity do:** After each task: (1) spec compliance review, (2) code quality review. Both automated via sub-agents.

**Recommendation:** Keep current approach (separate review phase = Phase 7). Two-stage review per task is overkill for your framework — it's a Phase 7 concern, not Phase 4.

### 4. No Goal-Backward Verification (must_haves)

**Current:** Acceptance criteria verify "did we build what was specified" but not "does the user's goal work end-to-end."

**GSD does:**
```yaml
must_haves:
  truths:
    - "Client can browse available carriers with filtering"
    - "Client can create a contract with encrypted credentials"
  artifacts:
    - path: "src/.../CarriersController.cs"
      provides: "Carrier catalog API"
  key_links:
    - from: "CarriersController"
      to: "CarrierCatalogService"
      via: "DI injection"
```

**Recommendation:** Add a `Goal Verification` section per story that defines observable user behavior, not just technical acceptance criteria.

### 5. No Story-Level Wave Plan Within Epics

**Current:** Waves at epic level only. Within Epic 04 (10 stories), all stories appear sequential.

**But:** Story 4.2 (Carriers) and Story 4.3 (Contracts) are independent — they could run in parallel.

**Recommendation:** Add wave groups within each epic for stories that can run in parallel.

---

## Proposed Phase 4 Design

Based on the comparison, your existing structure is already 80% there. Phase 4 should formalize it with a few additions:

### 6 Steps

| Step | Name | Gate? |
|------|------|-------|
| 1 | Context (silent) — read Phase 3 design | - |
| 2 | Epic Definition — goal, scope, acceptance criteria | - |
| 3 | Story Decomposition — stories with dependencies, priorities | - |
| 4 | Sub-task Breakdown — per story, with file paths and verification | - |
| 5 | Wave Plan — parallel groups within stories | - |
| 6 | Approval + Document | **HARD GATE** |

### Output Artifacts

```
04-decomposition/
├── index.md                    # Epic overview + wave plan + dependency diagram
├── epic.md                     # Epic definition with goal verification
├── stories/
│   ├── story-01-{slug}.md      # Story + sub-tasks + acceptance criteria
│   ├── story-02-{slug}.md
│   └── ...
└── wave-plan.md                # Which stories/tasks can run in parallel
```

### Sub-task Format (enhanced from your existing)

```markdown
- [ ] 4.1.1 Create `ApiResponse<T>` generic wrapper class
  - **File:** `src/CarrierPlatform.Core/Models/Api/ApiResponse.cs` (create)
  - **Action:** Create class with properties: bool Success, T? Data, ApiMeta? Meta, ApiError? Error
  - **Verify:** `dotnet build` compiles, type exists in namespace
  - **Depends on:** — (none)
```

vs your current:
```markdown
- [ ] 4.1.1 Create `ApiResponse<T>` generic wrapper class in
      `CarrierPlatform.Core/Models/Api/` with properties...
```

**Difference:** Added structured `File:`, `Verify:`, `Depends on:` fields. The description stays the same.

### Story Format (enhanced)

```markdown
### Story 4.2: Carrier Catalog Endpoints
**Priority:** P0 | **Estimate:** 4h | **Wave:** A (parallel with 4.3)
**Spec:** spec/03_api_specification.md (Section 4.2)
**Depends on:** Story 4.1 (API Infrastructure)

> Description...

**Goal Verification:**
- User can browse carriers filtered by country
- User can view carrier detail with services and parameters

**Sub-tasks:**
- [ ] 4.2.1 ...

**Acceptance Criteria:**
- [ ] ...
```

**Additions vs current:** `Wave:`, `Depends on:`, `Goal Verification:` fields.

### Wave Plan Format

```markdown
# Wave Plan: Epic 04 — Core API

## Wave A (no dependencies within epic)
- Story 4.1: API Infrastructure (foundation — blocks all others)

## Wave B (after Wave A)
- Story 4.2: Carrier Catalog ← (4.1)
- Story 4.3: Carrier Contracts ← (4.1)
- Story 4.9: Predefined Addresses ← (4.1)

## Wave C (after Wave B)
- Story 4.4: Delivery Endpoints ← (4.2, 4.3)
- Story 4.6: Tracking Endpoint ← (4.2)

## Wave D (after Wave C)
- Story 4.5: Shipment Endpoints ← (4.4)
- Story 4.7: Webhook Receiver ← (4.2)
- Story 4.8: Admin Endpoints ← (4.2, 4.3)

## Wave E (after Wave D)
- Story 4.10: OpenAPI Documentation ← (all)
```
