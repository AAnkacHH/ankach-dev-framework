---
name: review
description: >
  Phase 7: Three-stage code review — spec compliance, code quality, test verification.
  Plus manual review summary. Can loop back to Phase 6 for fixes or Phase 4/5 for redesign.
  MUST complete before Phase 8: Documentation.
---

# Phase 7: Review

<HARD-GATE>
Requires completed Phase 6 Implementation.
Read `.planning/active/06-implementation/index.md` — if not COMPLETE, stop and finish Phase 6 first.
</HARD-GATE>

## Staged Review Order (fail-fast)

```
Spec Compliance → Code Quality → Test Verification → Manual Review Summary
```

Each stage must PASS before the next runs. Don't waste time reviewing quality of code that doesn't meet spec.

---

## Step 1: Context

Read:
1. `.planning/active/03-design/` — what was designed (spec to check against)
2. `.planning/active/04-decomposition/` — stories and acceptance criteria
3. `.planning/active/06-implementation/` — agent reports
4. Actual code changes: `git diff main...HEAD`

**Critical rule: Read CODE, not agent reports.** Agent reports may be optimistic or incomplete. Always verify by reading the actual implementation.

---

## Step 2: Spec Compliance

Spawn `spec-reviewer` sub-agent (`.claude/agents/spec-reviewer.md`) to perform this check. The sub-agent is READ-ONLY and will verify code against Phase 3 design independently.

Check that implementation matches Phase 3 design — nothing more, nothing less.

**For each requirement from Phase 3:**

| Requirement | Implemented? | Where | Notes |
|-------------|-------------|-------|-------|
| {entity from data-model.md} | YES/NO/PARTIAL | `src/.../File.cs:L42` | {details} |
| {endpoint from api-contracts.md} | YES/NO/PARTIAL | `src/.../Controller.cs` | {details} |

**Check for:**
- [ ] Missing requirements — designed but not implemented
- [ ] Extra/unneeded work — implemented but not designed (scope creep)
- [ ] Misunderstandings — implemented differently than designed

Write to `.planning/active/07-review/spec-compliance.md`.

---

## Step 3: Code Quality

Spawn `code-quality-reviewer` sub-agent (`.claude/agents/code-quality-reviewer.md`) to perform this check. Only dispatch AFTER spec compliance passes.

Check implementation quality against project conventions.

**Categories and severity:**

| Category | CRITICAL (must fix) | HIGH (should fix) | MEDIUM | LOW |
|----------|--------------------|--------------------|--------|-----|
| **Security** | Credentials in DTOs, SQL injection, missing auth | XSS risk, weak validation | Logging sensitive data | Minor input sanitization |
| **Architecture** | DbContext in controller, business logic in controller | Missing service layer, tight coupling | God service | Minor naming |
| **Performance** | N+1 queries, sync where async needed | Missing indexes, unnecessary DB calls | Large response payloads | Minor optimization |
| **Conventions** | Violated AGENTS.md forbidden rules | Wrong patterns | Inconsistent naming | Style nitpicks |
| **Error Handling** | No error handling for external calls | Missing validation | Generic error messages | Missing edge cases |
| **File Quality** | >800 lines, >4 nesting levels | >50 line functions | Complex conditionals | Minor readability |

Write to `.planning/active/07-review/code-quality.md`.

---

## Step 4: Test Verification

Run tests and verify coverage.

```bash
dotnet test --verbosity normal
```

**Check:**
- [ ] All tests pass
- [ ] New code has tests
- [ ] Tests test LOGIC, not mocks (test real behavior)
- [ ] Edge cases covered (null, empty, invalid input, boundary values)
- [ ] Error scenarios tested

Write to `.planning/active/07-review/test-results.md`:

```markdown
# Test Results

## Test Run
- Total: {N}
- Passed: {N}
- Failed: {N}
- Skipped: {N}

## New Tests Added
- {test}: {what it verifies}

## Coverage Notes
- {areas well covered}
- {areas needing more tests}
```

---

## Step 5: Manual Review Summary

Prepare a concise summary for the user with:

```markdown
## Review Summary for {Feature Name}

### Verdict: SHIP / NEEDS WORK / BLOCKED

### Spec Compliance
{N}/{M} requirements implemented. {issues if any}

### Code Quality
{N} findings: {critical} CRITICAL, {high} HIGH, {medium} MEDIUM, {low} LOW

### Tests
{pass/fail}, {N} new tests added

### Key Findings (if any)
1. {most important finding}
2. {second most important}

### Recommendation
{SHIP: no critical/high issues}
{NEEDS WORK: list what to fix, recommend back to Phase 6}
{BLOCKED: architectural problem, recommend back to Phase 4/5}
```

---

## Step 6: Approval

Present the manual review summary to user.

```
Review complete. Verdict: {SHIP / NEEDS WORK / BLOCKED}
{summary}

`approve` → Phase 8: Documentation
`reject` → back to Phase 6 (fix) or Phase 4/5 (redesign)
```

### Write `.planning/active/07-review/index.md`:

```markdown
# Phase 7: Review — {Feature Name}

| Field    | Value                    |
|----------|--------------------------|
| Phase    | 7 of 9                  |
| Verdict  | SHIP / NEEDS WORK / BLOCKED |
| Date     | {YYYY-MM-DD}            |

## Summary
{2-3 sentences: review results}

## Spec Compliance
{N}/{M} requirements verified. {PASS/FAIL}

## Code Quality
{N} findings: {critical} CRITICAL, {high} HIGH, {medium} MEDIUM, {low} LOW

## Tests
{pass/fail}, {N} tests, {coverage}%

## Files in This Phase
- [spec-compliance.md](./spec-compliance.md)
- [code-quality.md](./code-quality.md)
- [test-results.md](./test-results.md)
- [revision-log.md](./revision-log.md) (if fixes applied)

## Gate
{Verdict} → Phase 8: Documentation / Back to Phase 6 / Back to Phase 4/5
```

<HARD-GATE>
Wait for explicit approval.
On SHIP + approve → update STATE.md, proceed to Phase 8.
On NEEDS WORK → user decides: back to Phase 6 for fixes, then re-review (max 2 iterations).
On BLOCKED → user decides: back to Phase 4/5 for redesign.
</HARD-GATE>

If fixes are made, write `.planning/active/07-review/revision-log.md` tracking what changed.

---

## Exit Criteria (ALL must be true)

- [ ] Spec compliance checked: every requirement verified against code (Step 2)
- [ ] Code quality reviewed with severity ratings (Step 3)
- [ ] Tests run and results documented (Step 4)
- [ ] Manual review summary prepared (Step 5)
- [ ] Verdict is SHIP (or issues fixed within max 2 iterations)
- [ ] All artifacts written to `.planning/active/07-review/`
- [ ] User approved
- [ ] STATE.md updated

---

## Anti-Rationalization Table

| Excuse | Reality |
|--------|---------|
| "The agent report says everything is fine" | Do NOT trust the report. Read the code yourself. |
| "Tests pass, so the code is correct" | Tests passing ≠ spec compliance. Check requirements. |
| "Minor issues can be fixed later" | Minor issues compound. Fix now or document explicitly. |
| "Review is taking too long, let's ship" | Shipping unreviewed code = shipping bugs to production. |
