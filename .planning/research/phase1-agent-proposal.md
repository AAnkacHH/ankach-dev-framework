# Phase 1: Analysis Agent — Design Proposal v2

**Date:** 2026-03-28
**Status:** DRAFT — awaiting review

---

## Architecture

```
User: /analyze Add bulk shipment endpoint
            │
            ▼
.claude/commands/analyze.md              ← thin trigger
            │
            ▼
.claude/skills/analysis/SKILL.md         ← 6-step workflow
            │
            ├── reads: CLAUDE.md, AGENTS.md, .context/*.md
            ├── reads: .planning/STATE.md (if exists)
            │
            ├── [brownfield] explores codebase directly
            ├── [greenfield] spawns: researcher sub-agent
            │
            ├── writes: .planning/active/01-analysis/*
            │
            └── HARD GATE → user approves → Phase 2
```

## Files

```
.claude/
├── commands/analyze.md                   # /analyze trigger
├── skills/analysis/SKILL.md             # 6-step workflow
└── agents/researcher.md                  # Research sub-agent (greenfield + deep research)

Output → .planning/active/01-analysis/
├── index.md                              # Main analysis doc + decision log
├── codebase-research.md                  # Brownfield: existing code findings
└── external-research.md                  # Greenfield/optional: external sources
```

---

## 6-Step Workflow

### Step 1: Context (silent)

Before any interaction, read and internalize:

1. `CLAUDE.md` — project rules, stack
2. `AGENTS.md` — architecture rules, forbidden patterns
3. `.context/project-structure.md` — directory layout
4. `.context/coding-conventions.md` — patterns
5. `.context/security-rules.md` — security constraints
6. `.planning/STATE.md` — if exists, check for active feature

**Detect project type:**

```
Has src/ or .sln or package.json or go.mod?
  YES → brownfield (existing project)
  NO  → greenfield (new project)
```

**Do not show this step to user. Internalize silently.**

---

### Step 2: Research

Automatically adapts to project type:

**Brownfield (existing project):**
- Identify affected layers (API / Core / Infrastructure / Carriers)
- Read existing code in affected area (controllers, services, repositories)
- Search for similar implementations (Grep for patterns)
- Check existing tests for related features
- Check recent git log (last 20 commits in affected area)
- Note existing patterns — new code MUST follow them
- Write to: `.planning/active/01-analysis/codebase-research.md`
- External research only if unknowns remain (new library, unfamiliar API)

**Greenfield (new project):**
- Spawn `researcher` sub-agent(s) for parallel research:
  - Stack decisions (frameworks, DB, libraries)
  - Architecture patterns for this domain
  - Similar projects / boilerplates
  - Known pitfalls
- Write to: `.planning/active/01-analysis/external-research.md`

**codebase-research.md template:**
```markdown
# Codebase Research: {Feature Name}

## Affected Layers
- API: {controllers}
- Core: {entities, interfaces, validators}
- Infrastructure: {repositories, services}

## Existing Patterns Found
- {Pattern}: see `src/.../File.cs:L42`

## Similar Implementations
- {Feature X} in `src/.../` — key difference: {what}

## Recent Relevant Changes
- {commit}: {description}

## Constraints Discovered
- {constraint from existing code}
```

**external-research.md template:**
```markdown
# External Research: {Feature Name}

## Sources Consulted
- {source 1}: {what was found}

## Key Findings
- {finding with confidence: HIGH/MEDIUM/LOW}

## Recommendations
- {recommendation based on research}
```

---

### Step 3: Questions

<RULES>
- ONE question per message. Never dump a list.
- Prefer MULTIPLE CHOICE (2-4 options with trade-offs).
- Show your assumption: "I'm assuming X. Correct, or would you prefer Y?"
- Max 7 questions. After 7, document remaining unknowns as assumptions.
- Skip questions answerable from codebase research.
- Follow energy — if user emphasizes something, dig deeper.
</RULES>

**Categories to check (skip if already clear):**
- [ ] Scope: included / excluded
- [ ] Users: who consumes this
- [ ] Data: entities, new or existing
- [ ] Behavior: edge cases, errors, validation
- [ ] Integration: external APIs, async, events
- [ ] Non-functional: performance, security, scale

**Anti-rationalization:**

| Excuse | Reality |
|--------|---------|
| "Requirements are clear enough" | Clear to you != clear to agent. Verify. |
| "User seems impatient" | 3 good questions now saves 3 rejected implementations later. |
| "I can infer this from code" | Inference != confirmation. Ask when it matters. |

---

### Step 4: Understanding Lock

<HARD-GATE>
Before proposing ANY approach, present this summary and get explicit confirmation.
Do NOT proceed without it.
</HARD-GATE>

```markdown
## Understanding Summary

- **What:** {what we're building}
- **Why:** {problem or value}
- **Who:** {who uses this}
- **Scope:** {included}
- **Not included:** {explicit non-goals}
- **Constraints:** {technical, security, performance}

## Assumptions
1. {assumption}
2. ...

> Does this accurately reflect your intent?
> Confirm or correct before I propose approaches.
```

**Acceptable:** "yes", "correct", "confirmed", "looks good"
**NOT acceptable:** silence, "interesting", "maybe"
**If corrections:** update, re-confirm.

---

### Step 5: Approaches

Present **2-3 approaches:**

```markdown
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
- Always include "simplest thing that works"
- Approaches must be genuinely different
- If one is clearly superior — say so
- Reference existing codebase patterns when possible

---

### Step 6: Approval + Document

User picks approach. Then write everything and get final approval.

**On user choice ("go with B"):**

1. Write `.planning/active/01-analysis/index.md`:

```markdown
# Phase 1: Analysis — {Feature Name}

| Field    | Value                    |
|----------|--------------------------|
| Phase    | 1 of 9                  |
| Status   | APPROVED                |
| Date     | {YYYY-MM-DD}            |

## Summary
{3-5 sentences}

## Context
{What prompted this work}

## Current State
{What exists, affected areas — or "Greenfield" if new project}

## Requirements
### Must Have
- {requirement}
### Nice to Have
- {requirement}
### Non-Goals
- {excluded}

## Chosen Approach: {Name}
{Description, 1-2 paragraphs}

## Risk Assessment
- **Level:** Low / Medium / High
- **Risks:** {what could go wrong}
- **Mitigations:** {how to address}

## Decision Log
### Approach Selection
- **Decided:** Approach {X}: {Name}
- **Alternatives:** {rejected with brief reason}
- **Rationale:** {why chosen}

### {Other decisions from Q&A}
- **Decided:** ...
- **Rationale:** ...

## Assumptions
1. {assumption}

## Open Questions for Phase 2
- {deferred}

## Files in This Phase
- [codebase-research.md](./codebase-research.md)
- [external-research.md](./external-research.md)

## Gate
Approved → Phase 2: Specification
```

2. **Self-review** (4 checks):
   - Placeholder scan: any TBD/TODO?
   - Consistency: sections contradict?
   - Scope: focused enough?
   - Ambiguity: could requirements be read two ways?

3. Present to user:

```
Analysis saved to `.planning/active/01-analysis/`.
Chosen approach: {Name} — {1 sentence}

Please review. `approve` → Phase 2 | `reject` → tell me what to fix.
```

<HARD-GATE>
Wait for explicit approval. On approve → update STATE.md. On reject → fix and re-present.
</HARD-GATE>

---

## Exit Criteria (ALL must be true)

- [ ] Project context read (Step 1)
- [ ] Research done — codebase and/or external (Step 2)
- [ ] Critical questions answered or documented as assumptions (Step 3)
- [ ] Understanding Lock confirmed by user (Step 4)
- [ ] 2-3 approaches proposed (Step 5)
- [ ] User approved approach + final document (Step 6)
- [ ] STATE.md updated

---

## Agent Definition: `.claude/agents/researcher.md`

```yaml
---
name: researcher
description: >
  Research sub-agent for analysis phases. Explores codebase patterns and external
  documentation. Spawned for deep research when main session needs background work.
tools: Read, Write, Grep, Glob, Bash, WebSearch, WebFetch
model: sonnet
maxTurns: 15
---
```

Researcher is used for:
- Greenfield: parallel domain research (stack, patterns, pitfalls)
- Brownfield: deep codebase exploration when main session is busy with conversation
- Any phase: external documentation lookup

---

## Command: `.claude/commands/analyze.md`

```markdown
---
description: "Phase 1: Analysis. Start here for any new feature, change, or investigation."
---

Read and follow `.claude/skills/analysis/SKILL.md` step by step.

User request: $ARGUMENTS

Rules:
- Follow EVERY step. Do not skip.
- No code. Analysis only.
- Wait for approval at every HARD GATE.
- Write artifacts to .planning/active/01-analysis/
```
