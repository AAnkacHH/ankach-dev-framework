---
name: implementation
description: >
  Phase 6: Parallel agent implementation. Execute stories wave-by-wave with fresh
  context per story, deviation rules, atomic commits. Use after Phase 5 Validation passes.
  MUST complete before Phase 7: Review.
---

# Phase 6: Implementation

<HARD-GATE>
Requires approved Phase 5 Validation (PASS).
Read `.planning/active/05-validation/index.md` — if not PASS, stop and resolve validation issues first.
</HARD-GATE>

---

## Step 1: Context (Silent)

Read:
1. `.planning/active/04-decomposition/wave-plan.md` — execution order
2. `.planning/active/04-decomposition/stories/` — all story files
3. `.planning/active/05-validation/index.md` — confirm PASS
4. `CLAUDE.md` + `AGENTS.md` — project rules
5. `.planning/STATE.md` — verify Phase 6

---

## Step 2: Setup

- Create git branch: `feature/{slug}` (from STATE.md)
- Run baseline tests: `dotnet test` — all must pass before starting
- Create `.planning/active/06-implementation/` directory

<HARD-GATE>
If baseline tests fail, STOP. Do not implement on a broken baseline.
Report failures to user.
</HARD-GATE>

---

## Step 3: Execute Waves

For each wave (A, B, C...):

### Per Story in Wave

Dispatch fresh `implementer` sub-agent (`.claude/agents/implementer.md`) per story with:
- Full story text (sub-tasks, acceptance criteria)
- Relevant Phase 3 design files (data-model.md, api-contracts.md)
- CLAUDE.md + AGENTS.md rules

**Agent receives these instructions:**
1. Implement each sub-task in order
2. Follow existing codebase patterns exactly
3. Atomic git commit per sub-task
4. Report status when done

**Deviation Rules:**

| Rule | Trigger | Action | Limit |
|------|---------|--------|-------|
| Rule 1 (Bug) | Code doesn't work as planned | Auto-fix | 3 attempts |
| Rule 2 (Missing Critical) | No validation, auth, error handling | Auto-add | 3 attempts |
| Rule 3 (Blocking) | Missing deps, config, imports | Auto-resolve | 3 attempts |
| Rule 4 (Architectural) | Needs new table, major restructuring | **STOP — ask user** | N/A |

**Agent Status Codes:**
- `DONE` — all sub-tasks complete, tests pass
- `DONE_WITH_CONCERNS` — complete but has doubts (review before proceeding)
- `BLOCKED` — cannot complete, needs help
- `NEEDS_CONTEXT` — missing information

**Guards:**
- Analysis paralysis: 5+ file reads without any writes = STOP, report blocker
- Scope boundary: only fix issues caused by current task. Pre-existing issues → `issues.md`

### Per Story Report

Write `.planning/active/06-implementation/agents/wave-{X}-story-{NN}.md`:

```markdown
# Implementation: Story {N}.{M} — {Name}

**Status:** DONE / DONE_WITH_CONCERNS / BLOCKED / NEEDS_CONTEXT
**Agent:** implementer
**Wave:** {X}

## Completed Sub-tasks
- [x] {N}.{M}.1 — {what was done}
- [x] {N}.{M}.2 — {what was done}

## Files Changed
- `src/.../File.cs` — created / modified

## Commits
- `{hash}` — {message}

## Concerns (if any)
- {concern}

## Deviations (if any)
- [Rule {N}] {what deviated and why}
```

---

## Step 4: Wave Report

After each wave completes, present to user:

```
Wave {X} complete. {N}/{M} stories done.
- Story {N}.1: DONE
- Story {N}.2: DONE_WITH_CONCERNS — {concern summary}

Next: Wave {Y} ({K} stories). Proceed?
```

<GATE>
User approves next wave. This is NOT a hard gate — user can say "continue all" to skip per-wave approval.
</GATE>

---

## Step 5: Implementation Summary

After all waves complete, write `.planning/active/06-implementation/index.md`:

```markdown
# Phase 6: Implementation — {Feature Name}

| Field    | Value                    |
|----------|--------------------------|
| Phase    | 6 of 9                  |
| Status   | COMPLETE                |
| Date     | {YYYY-MM-DD}            |

## Summary
{N} stories implemented across {K} waves. {M} total commits.

## Wave Progress
| Wave | Stories | Status |
|------|---------|--------|
| A | {N}.1 | DONE |
| B | {N}.2, {N}.3 | DONE |

## Issues
{Link to issues.md if any deviations or concerns}

## Files in This Phase
- [agents/wave-A-story-01.md](./agents/wave-A-story-01.md)
- ...

## Gate
Ready for Phase 7: Review
```

---

## Step 6: Approval

```
Implementation complete. {N} stories, {M} commits, {K} waves.
All agent reports in `.planning/active/06-implementation/agents/`.

`approve` → Phase 7: Review | `reject` → tell me what to fix.
```

<HARD-GATE>
Wait for approval. On approve → update STATE.md.
</HARD-GATE>

---

## Exit Criteria (ALL must be true)

- [ ] Git branch created and baseline tests pass (Step 2)
- [ ] All waves executed, all stories have agent reports (Step 3)
- [ ] No BLOCKED stories without resolution (Step 3)
- [ ] Implementation summary written (Step 5)
- [ ] All artifacts written to `.planning/active/06-implementation/`
- [ ] User approved
- [ ] STATE.md updated

---

## Anti-Rationalization Table

| Excuse | Reality |
|--------|---------|
| "I can implement everything in one go" | Fresh context per story prevents pollution and errors. |
| "Wave ordering doesn't matter" | Dependencies exist. Ignoring them causes conflicts. |
| "Tests can wait until later" | Tests written later don't catch the bugs you introduced now. |
| "This deviation is fine, no need to report" | Unreported deviations accumulate and break the review phase. |
