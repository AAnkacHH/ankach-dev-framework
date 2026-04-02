# Agents

Five specialized sub-agents with scoped tool access.

## Overview

| Agent | Phase | Tools | Model | Role |
|-------|-------|-------|-------|------|
| `researcher` | 1, 2 | Read, Write, Grep, Glob, Bash, WebSearch, WebFetch | Sonnet | Codebase + external research |
| `implementer` | 6 | Read, Write, Edit, Grep, Glob, Bash | Sonnet | Implement a single story |
| `test-writer` | 6 | Read, Write, Edit, Grep, Glob, Bash | Sonnet | Write unit + integration tests |
| `spec-reviewer` | 7 | Read, Grep, Glob | Sonnet | Spec compliance (READ-ONLY) |
| `code-quality-reviewer` | 7 | Read, Grep, Glob, Bash | Opus | Code quality review (READ-ONLY) |

> Reviewers are deliberately **read-only** — they cannot modify code, only report findings.

---

## Researcher

**File:** `.claude/agents/researcher.md`
**Used in:** Phase 1 (Analysis), Phase 2 (Architecture), Phase 5 (Validation)

Gathers information from codebase and external sources. Does NOT make decisions.

- **Greenfield:** parallel domain research (stack, patterns, pitfalls)
- **Brownfield:** deep codebase exploration, pattern analysis
- **External:** library docs (context7 MCP), API docs, best practices

**Rules:**
- Cite sources for every finding (file path, URL, doc section)
- Confidence levels: HIGH (official docs), MEDIUM (verified search), LOW (unverified)
- Max 15 turns — report what you have
- Can only write to `.planning/` files, not source code

---

## Implementer

**File:** `.claude/agents/implementer.md`
**Used in:** Phase 6 (Implementation)

Receives a single story with sub-tasks and implements them. Fresh context per story.

**Workflow:**
1. Read story + design files + project rules
2. Implement each sub-task in order
3. Follow existing codebase patterns (Grep before writing)
4. Atomic git commit per sub-task
5. Run tests after each sub-task
6. Report status

**Deviation Rules:**

| Rule | Trigger | Action | Limit |
|------|---------|--------|-------|
| Rule 1 | Bug — code doesn't work | Auto-fix | 3 attempts |
| Rule 2 | Missing validation/auth/error handling | Auto-add | 3 attempts |
| Rule 3 | Missing deps, broken imports | Auto-resolve | 3 attempts |
| Rule 4 | Architectural decision needed | **STOP — report to orchestrator** | N/A |

**Status Codes:**
- `DONE` — all sub-tasks complete, tests pass
- `DONE_WITH_CONCERNS` — complete but has doubts
- `BLOCKED` — cannot complete, needs help
- `NEEDS_CONTEXT` — missing information

**Guards:**
- Analysis paralysis: 5+ file reads without writes = STOP
- Scope boundary: only fix current task issues, pre-existing → `issues.md`

---

## Test Writer

**File:** `.claude/agents/test-writer.md`
**Used in:** Phase 6 (Implementation)

Writes tests following project conventions.

**Pattern:** `{Method}_{Scenario}_{ExpectedResult}`
**Framework:** xUnit + NSubstitute + FluentAssertions
**Helper:** `FakeCurrentUser` from `tests/Helpers/`

**Categories:**
- Unit tests: service logic, validators, mappers
- Integration tests: repository queries, API endpoints

**Rules:**
- Follow existing test patterns (Grep for examples)
- Don't modify production code
- Cover edge cases: null, empty, invalid, boundary
- Tests test LOGIC, not mocks
- Use `DateTimeOffset.UtcNow`, never `DateTime.Now`

---

## Spec Reviewer

**File:** `.claude/agents/spec-reviewer.md`
**Used in:** Phase 7 (Review)

Verifies implementation matches Phase 3 design — nothing more, nothing less.

**Critical rule:** "Do NOT trust the implementer's report. Read actual CODE."

**Checks:**
1. Missing requirements — designed but not implemented
2. Extra/unneeded work — implemented but not designed (scope creep)
3. Misunderstandings — implemented differently than designed

**Output:** PASS or FAIL with file:line references for each issue.

**Tools:** Read, Grep, Glob only — **cannot modify any files**.

---

## Code Quality Reviewer

**File:** `.claude/agents/code-quality-reviewer.md`
**Used in:** Phase 7 (Review) — only dispatched AFTER spec compliance passes

**Model:** Opus (strongest reasoning for quality assessment)

**Severity levels:**

| Level | Examples |
|-------|---------|
| CRITICAL | Credentials in DTOs, SQL injection, missing auth |
| HIGH | >800 line files, >4 nesting, missing error handling |
| MEDIUM | Inefficient queries, unnecessary complexity |
| LOW | Naming style, minor readability |

**Verdict:**
- `APPROVE` — 0 critical + 0 high issues
- `NEEDS WORK` — issues found, fixable
- `BLOCK` — architectural problem

**Tools:** Read, Grep, Glob, Bash (for `git diff`, `dotnet test`) — **cannot modify files**.
