---
name: validation
description: >
  Phase 5: Validate architecture, requirements coverage, story completeness before
  implementation. Architecture review for deprecated tech, anti-patterns, security.
  Can loop back to Phase 3/4. MUST complete before Phase 6: Implementation.
---

# Phase 5: Validation

<HARD-GATE>
Requires approved Phase 4 Decomposition.
Read `.planning/active/04-decomposition/index.md` — if not APPROVED, stop and complete Phase 4 first.
</HARD-GATE>

## Purpose

Quality gate between planning and coding. Catches issues that would be expensive to fix during implementation. Max 2 revision iterations — then escalate to user.

---

## Step 1: Context (Silent)

Read all previous phase artifacts:
1. `.planning/active/02-architecture/tech-decisions.md` — ADRs
2. `.planning/active/03-design/` — domain map, data model, API contracts
3. `.planning/active/04-decomposition/` — epic, stories, wave plan
4. `CLAUDE.md` + `AGENTS.md` — project conventions

---

## Step 2: Architecture Review

Check tech decisions from Phase 2-3 for issues. Spawn `researcher` sub-agent for version/compatibility checks if needed.

| Category | What to Check |
|----------|---------------|
| **Deprecated/EOL** | Technologies no longer supported or approaching EOL |
| **Anti-patterns** | God service, tight coupling, N+1, anemic models |
| **Security** | Credentials in DTOs, missing auth, SQL injection risk, XSS |
| **Performance** | Sync where async needed, missing indexes, no caching strategy |
| **Over-engineering** | CQRS for CRUD, microservices for monolith task, unnecessary abstraction |
| **Under-engineering** | No error handling, no retry for external APIs, no validation |
| **Framework misuse** | `HasDefaultValue(enum)` in EF, DbContext in controller, `.Now` instead of `.UtcNow` |
| **Compatibility** | Library doesn't support current runtime, breaking changes |

**Findings severity:**
- **BLOCKER** — must fix before implementation
- **WARNING** — should fix, but can proceed
- **INFO** — consider, non-blocking

---

## Step 3: Requirements Coverage

Verify every requirement from Phase 3 maps to a story in Phase 4:

```markdown
| Requirement | Phase 3 Source | Phase 4 Story | Status |
|-------------|---------------|---------------|--------|
| {requirement} | data-model.md: {entity} | Story {N}.{M} | COVERED / MISSING |
```

Any MISSING = FAIL.

---

## Step 4: Story Completeness + Alignment + Scope + Placeholders

**Story Completeness** — each story has:
- [ ] Acceptance criteria (testable, not vague)
- [ ] `Depends on:` field
- [ ] `Wave:` field
- [ ] Sub-tasks with file paths

**Design Alignment** — decomposition matches Phase 3:
- [ ] Entities from data-model.md appear in sub-tasks
- [ ] Endpoints from api-contracts.md appear in sub-tasks
- [ ] Domain map subdomains align with story grouping

**Scope Sanity:**
- [ ] No stories/sub-tasks that weren't in Phase 1-3 scope (scope creep)
- [ ] No missing stories for approved requirements

**No Placeholders:**
- [ ] No "TBD", "TODO", "implement later" in sub-tasks
- [ ] No vague acceptance criteria ("works correctly", "looks good")

---

## Step 5: Validation Report

Write `.planning/active/05-validation/index.md`:

```markdown
# Phase 5: Validation — {Feature Name}

| Field    | Value                    |
|----------|--------------------------|
| Phase    | 5 of 9                  |
| Status   | PASS / FAIL              |
| Date     | {YYYY-MM-DD}            |

## Architecture Review
{N} findings: {blockers} BLOCKER, {warnings} WARNING, {info} INFO

## Requirements Coverage
{N}/{M} requirements covered ({percentage}%)

## Story Completeness
{pass/fail per check}

## Verdict
PASS — ready for Phase 6 / FAIL — see issues below
```

Write detailed findings to:
- `.planning/active/05-validation/architecture-review.md` — tech review findings
- `.planning/active/05-validation/checklist.md` — coverage + completeness checks
- `.planning/active/05-validation/gaps.md` — specific issues (if any)

---

## Step 6: Approval

Present to user:

```
Validation complete. Result: {PASS/FAIL}
{summary of findings}

`approve` → Phase 6: Implementation | `reject` → back to Phase {3/4} to fix
```

<HARD-GATE>
On PASS + approve → update STATE.md, proceed to Phase 6.
On FAIL → list issues, recommend which phase to revisit (3 or 4), wait for user decision.
Max 2 revision loops. After 2 failures, escalate: user decides to force proceed or redesign.
</HARD-GATE>

### On FAIL and loop-back:

Write `.planning/active/05-validation/revision-log.md`:

```markdown
# Revision Log

## Iteration {N}
**Date:** {YYYY-MM-DD}
**Result:** FAIL
**Issues:** {list of issues found}
**Action:** Revisit Phase {3/4} to fix: {specific items}
**Fixed:** {what was changed after revisit}
```

---

## Exit Criteria (ALL must be true)

- [ ] Architecture review completed with severity ratings (Step 2)
- [ ] Requirements coverage: every Phase 3 requirement mapped to Phase 4 story (Step 3)
- [ ] Story completeness verified (Step 4)
- [ ] Alignment, scope, and placeholder checks passed (Step 4)
- [ ] Validation report written with PASS/FAIL (Step 5)
- [ ] All artifacts written to `.planning/active/05-validation/`
- [ ] User approved (or escalated after max 2 iterations)
- [ ] STATE.md updated

---

## Anti-Rationalization Table

| Excuse | Reality |
|--------|---------|
| "Everything looks fine, let's just start coding" | That's exactly when validation catches hidden gaps. |
| "Architecture review is overkill" | One deprecated library found now saves days of refactoring later. |
| "Requirements are obviously covered" | Obvious ≠ verified. Check the mapping. |
| "Validation is just bureaucracy" | Implementation without validation = guaranteed rework. |
