---
name: analysis
description: >
  Use BEFORE any implementation — new features, changes, refactoring, bug investigation.
  Explores context, researches codebase/external, asks questions, proposes approaches.
  MUST complete and be approved before Phase 2: Architecture.
---

# Phase 1: Analysis & Brainstorm

<HARD-GATE>
No code, no scaffolding, no file creation, no implementation until this phase is complete
and the user explicitly approves. This applies to EVERY task regardless of perceived simplicity.
</HARD-GATE>

## Anti-Pattern: "This Is Too Simple To Need Analysis"

Every task goes through this process. A single endpoint, a config change, a bug fix — all of them.
"Simple" tasks are where unexamined assumptions cause the most wasted work.
The analysis can be short (a few sentences for truly simple tasks), but you MUST complete all steps.

---

## Step 1: Context (Silent)

Before any interaction with the user, read and internalize these files:

1. `CLAUDE.md` — project rules, stack, conventions
2. `AGENTS.md` — architecture rules, forbidden patterns
3. `.context/project-structure.md` — directory layout, file naming
4. `.context/coding-conventions.md` — patterns, code style
5. `.context/security-rules.md` — security constraints
6. `.planning/STATE.md` — if exists, check for active feature

**Detect project type:**

```
Has src/ or .sln or package.json or go.mod or pyproject.toml?
  YES → brownfield (existing project)
  NO  → greenfield (new project)
```

**If STATE.md shows an active feature:**
Warn the user: "There is an active feature in progress: {name}, Phase {N}. Start a new analysis anyway?"
Wait for confirmation before proceeding.

**Do not show this step to the user. Internalize context silently.**

---

## Step 2: Research

Automatically adapt to project type.

**Before writing any research files, create the output directory:**
`mkdir -p .planning/active/01-analysis/`

### Brownfield (existing project)

Research the existing codebase in the area relevant to the request:

- [ ] Identify affected layers (API / Core / Infrastructure / Carriers / Tests)
- [ ] Read existing code in the affected area (controllers, services, repositories, entities)
- [ ] Search for similar implementations (`Grep` for related patterns)
- [ ] Check existing tests for related features
- [ ] Check recent git history: `git log --oneline -20 -- {affected paths}`
- [ ] Note existing patterns — new code MUST follow them exactly

Write findings to `.planning/active/01-analysis/codebase-research.md` using this template:

```markdown
# Codebase Research: {Feature Name}

## Affected Layers
- API: {controllers}
- Core: {entities, interfaces, validators}
- Infrastructure: {repositories, services}
- Carriers: {if applicable}

## Existing Patterns Found
- {Pattern}: see `src/.../File.cs:L42`

## Similar Implementations
- {Feature X} in `src/.../` — key difference: {what's different}

## Recent Relevant Changes
- {commit hash}: {description}

## Constraints Discovered
- {constraint from existing code}
```

**External research** only if unknowns remain (new library, unfamiliar API, carrier integration).

### Greenfield (new project)

Spawn `researcher` sub-agent(s) for parallel domain research:
- Stack decisions (frameworks, DB, libraries for this domain)
- Architecture patterns for this type of project
- Similar projects / boilerplates
- Known pitfalls and anti-patterns

Write findings to `.planning/active/01-analysis/external-research.md` using this template:

```markdown
# External Research: {Feature Name}

## Sources Consulted
- {source}: {what was found}

## Key Findings
- {finding} — confidence: HIGH / MEDIUM / LOW

## Recommendations
- {recommendation based on research}
```

---

## Step 3: Questions

Ask clarifying questions to eliminate ambiguity.

<RULES>
- ONE question per message. Never dump a list of questions.
- Prefer MULTIPLE CHOICE (2-4 options with trade-offs) when possible.
- Show your assumption: "I'm assuming X. Correct, or would you prefer Y?"
- Max 7 questions. After 7, document remaining unknowns as assumptions.
- Skip questions answerable from codebase research (Step 2).
- Follow energy — if user emphasizes something, dig deeper there.
</RULES>

**Categories to check (skip if already clear from context/research):**
- [ ] **Scope:** What's included / excluded?
- [ ] **Users:** Who consumes this? (API clients, admin UI, internal)
- [ ] **Data:** What entities are involved? New or existing?
- [ ] **Behavior:** Edge cases, error handling, validation rules
- [ ] **Integration:** External APIs, async processing (Wolverine), events
- [ ] **Non-functional:** Performance requirements, security constraints, scale

**Anti-rationalization:**

| Excuse | Reality |
|--------|---------|
| "Requirements are clear enough" | Clear to you ≠ clear to the agent. Verify. |
| "User seems impatient" | 3 good questions now saves 3 rejected implementations later. |
| "I can infer this from code" | Inference ≠ confirmation. Ask when it matters. |
| "This is too simple for questions" | Simple tasks have hidden edge cases. Ask anyway. |

---

## Step 4: Understanding Lock

<HARD-GATE>
Before proposing ANY approach, present this summary and get explicit confirmation.
Do NOT proceed without it.
</HARD-GATE>

Present the following to the user:

```
## Understanding Summary

- **What:** {what we're building/changing, 1-2 sentences}
- **Why:** {problem being solved or value being added}
- **Who:** {who uses this — API consumers, admin, internal}
- **Scope:** {what's included}
- **Not included:** {explicit non-goals}
- **Constraints:** {technical, security, performance}

## Assumptions
1. {assumption — documented if not explicitly confirmed}
2. ...

> Does this accurately reflect your intent?
> Confirm or correct before I propose approaches.
```

**Acceptable confirmations:** "yes", "correct", "confirmed", "looks good", "so"
**NOT acceptable:** silence, "interesting", "hmm", "maybe"
**If corrections:** update summary, re-present, re-confirm.

---

## Step 5: Approaches

Present **2-3 concrete approaches** to the user.

For each approach:

```
### Approach A: {Name}

**How it works:** {2-3 sentences}
**Pros:** {concrete benefits}
**Cons:** {concrete drawbacks}
**Complexity:** Low / Medium / High
**Files affected:** {count + key files}
**Risk:** Low / Medium / High — {what could go wrong}
**Similar to:** {existing pattern in codebase, if any}
```

**Rules:**
- Always include "simplest thing that works" as one approach
- Approaches must be genuinely different, not cosmetic variations
- If one is clearly superior — say so, don't pretend they're equal
- Reference existing codebase patterns: "Similar to how {X} does {Y}"
- For brownfield: approaches must respect existing architecture

**Anti-rationalization:**

| Excuse | Reality |
|--------|---------|
| "There's only one way to do this" | There are always alternatives. Present at least 2. |
| "The user will pick what I recommend anyway" | Your recommendation may be wrong. Show options. |
| "All approaches are basically the same" | Then they differ in complexity, risk, or maintainability. Show that. |

---

## Step 6: Approval + Document

User picks approach. Write all artifacts and get final approval.

**On user choice ("go with B"):**

### 1. Write `.planning/active/01-analysis/index.md`:

```markdown
# Phase 1: Analysis — {Feature Name}

| Field    | Value                    |
|----------|--------------------------|
| Phase    | 1 of 9                  |
| Status   | APPROVED                |
| Date     | {YYYY-MM-DD}            |

## Summary
{3-5 sentences: what was analyzed, what was decided}

## Context
{What prompted this work, 2-3 sentences}

## Current State
{What exists in codebase, affected areas — or "Greenfield project" if new}

## Requirements
### Must Have
- {requirement}
### Nice to Have
- {requirement}
### Non-Goals
- {explicitly excluded}

## Chosen Approach: {Name}
{Description of chosen approach, 1-2 paragraphs}

## Risk Assessment
- **Level:** Low / Medium / High
- **Risks:** {what could go wrong}
- **Mitigations:** {how to address}

## Decision Log
### Approach Selection
- **Decided:** Approach {X}: {Name}
- **Alternatives:** {rejected approaches with brief reason}
- **Rationale:** {why this approach was chosen}

### {Other decisions made during Q&A}
- **Decided:** {decision}
- **Rationale:** {why}

## Assumptions
1. {assumption}
2. ...

## Open Questions for Phase 2
- {deferred to architecture phase}

## Files in This Phase
- [codebase-research.md](./codebase-research.md) — existing code findings
- [external-research.md](./external-research.md) — external sources (if applicable)

## Gate
Approved {YYYY-MM-DD}. Proceed to Phase 2: Architecture.
```

### 2. Self-review (4 checks)

Before presenting to user, review your own work:

1. **Placeholder scan:** Any "TBD", "TODO", incomplete sections, vague requirements? Fix them.
2. **Internal consistency:** Do sections contradict each other?
3. **Scope check:** Is this focused enough for one feature? Or does it need decomposition?
4. **Ambiguity check:** Could any requirement be interpreted two different ways? If so, pick one and make it explicit.

Fix any issues inline. Do not ask the user to review your self-review.

### 3. Present to user

```
Analysis saved to `.planning/active/01-analysis/`.
Chosen approach: {Name} — {1 sentence summary}

Please review. `approve` → Phase 2: Architecture | `reject` → tell me what to fix.
```

<HARD-GATE>
Wait for explicit approval before any Phase 2 work.
On approve → create/update .planning/STATE.md.
On reject → ask what to fix, update, re-present.
</HARD-GATE>

### 4. Update STATE.md

On approval, create or update `.planning/STATE.md`:

```markdown
# Feature State

| Field          | Value                    |
|----------------|--------------------------|
| Feature        | {Feature Name}           |
| Slug           | {feature-slug}           |
| Started        | {YYYY-MM-DD}             |
| Current Phase  | 2 — Architecture         |
| Status         | IN_PROGRESS              |
| Branch         | feature/{slug}           |
| Type           | brownfield / greenfield  |

## Phase Log

| Phase | Status    | Approved   | Attempts | Notes |
|-------|-----------|------------|----------|-------|
| 1     | completed | {date}     | 1        |       |
| 2     | pending   | —          | 0        |       |

## Decision Log

- [Phase 1] {key decision from analysis}
```

---

## Exit Criteria (ALL must be true)

- [ ] Project context read and understood (Step 1)
- [ ] Research done — codebase and/or external (Step 2)
- [ ] Critical questions answered or documented as assumptions (Step 3)
- [ ] Understanding Lock confirmed by user (Step 4)
- [ ] 2-3 approaches proposed with trade-offs (Step 5)
- [ ] User approved approach + final document (Step 6)
- [ ] All artifacts written to `.planning/active/01-analysis/`
- [ ] STATE.md created/updated

**If ANY criterion is unmet: do NOT proceed to Phase 2.**

---

## Anti-Rationalization Table

| Excuse | Reality |
|--------|---------|
| "This is too simple for analysis" | Simple tasks have hidden assumptions. 5 minutes of analysis prevents hours of rework. |
| "I already know the solution" | Your solution may miss edge cases. Document it anyway. |
| "Let me just write the code" | Code without analysis = rework. Always analyze first. |
| "Requirements are obvious" | Obvious to you ≠ obvious. Understanding Lock proves clarity. |
| "Just one small change" | Small changes in the wrong direction compound. Analyze. |
| "User wants it done fast" | 10 minutes of analysis saves 2 hours of wrong implementation. |
| "I can skip the codebase search" | Existing patterns dictate how new code must be written. Always check. |
| "Questions will annoy the user" | Wrong implementation annoys more than questions. Ask. |
