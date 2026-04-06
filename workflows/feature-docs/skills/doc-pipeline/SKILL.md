---
name: doc-pipeline
description: >
  Parallel documentation pipeline: analyze a feature, then spawn 3 agents simultaneously
  (requirements, use cases, diagrams) to generate documentation. Use when you need
  feature documentation WITHOUT implementation. For docs + code, use dev-pipeline instead.
---

# Feature Documentation Pipeline

<HARD-GATE>
This workflow produces DOCUMENTATION ONLY — no code, no scaffolding, no implementation.
If the user needs both docs and code, redirect them to `/ankach:workflow-feature`.
</HARD-GATE>

---

## Step 1: Context (Silent)

Before any interaction with the user, read and internalize these files:

1. `CLAUDE.md` — project rules, stack, conventions
2. `AGENTS.md` — architecture rules, forbidden patterns
3. `.context/tech-stack.md` — language, frameworks, DB engine, ORM, test runner
4. `.context/project-structure.md` — directory layout, file naming
5. `.planning/STATE.md` — if exists, check for active feature

**Detect project type:**

```
Has src/ or .sln or package.json or go.mod or pyproject.toml?
  YES → brownfield (existing project)
  NO  → greenfield (new project)
```

**Do not show this step to the user. Internalize context silently.**

---

## Step 2: Stack Confirmation

<HARD-GATE>
Do NOT proceed until the user confirms the tech stack.
Documentation must reference the correct technologies — wrong stack assumptions
produce requirements and diagrams that mislead future implementation.
</HARD-GATE>

### Brownfield (existing project)

If `.context/tech-stack.md` exists — read it and present the detected stack.
If `.context/tech-stack.md` does NOT exist — scan the project yourself:
- Read dependency/project files (package manifests, lock files, project configs)
- Check database configuration files and connection strings
- Scan source directories for framework-specific patterns
- Check for frontend assets, templates, or SPA framework files

Present to the user:

```
## Detected Tech Stack

| Layer       | Technology               |
|-------------|--------------------------|
| Language    | {language + version}     |
| Backend     | {framework + version}    |
| Database    | {engine + version}       |
| ORM / Data  | {ORM or data layer}     |
| Frontend    | {framework or template engine} |
| Test runner | {test framework}         |

> Is this correct? Confirm or correct before I continue.
```

**Rules:**
- Only include layers that apply — skip rows that don't exist
- If a layer is ambiguous — ask, do not assume
- If you cannot confidently determine a layer — ask directly, do not guess

**On confirmation:** proceed to Step 3.
**On correction:** update, re-present, re-confirm.

### Greenfield (new project)

If `.context/PROJECT.md` or `.context/tech-stack.md` exists — read and present the planned stack.
If no context files exist — ask the user for their stack preferences.

**On response:** present a summary table and confirm.

---

## Step 3: Research

Automatically adapt to project type.

### Brownfield

Research the existing codebase in the area relevant to the feature:

- [ ] Identify affected layers and components
- [ ] Read existing code in the affected area
- [ ] Search for similar implementations
- [ ] Note existing patterns that the feature must follow

### Greenfield

Research the domain:
- Similar features in other systems
- Best practices and known pitfalls
- Architecture patterns that apply

---

## Step 4: Questions

Ask clarifying questions to eliminate ambiguity.

<RULES>
- ONE question per message. Never dump a list.
- Prefer MULTIPLE CHOICE (2-4 options with trade-offs) when possible.
- Show your assumption: "I'm assuming X. Correct, or would you prefer Y?"
- Max 7 questions. After 7, document remaining unknowns as assumptions.
- Skip questions answerable from codebase research (Step 3).
</RULES>

**Categories to check (skip if already clear):**
- [ ] **Scope:** What's included / excluded?
- [ ] **Users:** Who consumes this?
- [ ] **Data:** What entities are involved? New or existing?
- [ ] **Behavior:** Edge cases, error handling, validation rules
- [ ] **Integration:** External APIs, events, async processing
- [ ] **Non-functional:** Performance, security, scale

---

## Step 5: Understanding Lock

<HARD-GATE>
Present this summary and get explicit confirmation before generating any documentation.
</HARD-GATE>

```
## Understanding Summary

- **What:** {what we're documenting, 1-2 sentences}
- **Why:** {problem being solved or value being added}
- **Who:** {who uses this}
- **Scope:** {what's included}
- **Not included:** {explicit non-goals}
- **Constraints:** {technical, security, performance}

## Assumptions
1. {assumption}
2. ...

> Does this accurately reflect your intent?
> Confirm or correct before I generate documentation.
```

**Acceptable confirmations:** "yes", "correct", "confirmed", "looks good", "so"
**If corrections:** update, re-present, re-confirm.

---

## Step 6: Parallel Generation

Create the output directory:
`mkdir -p .planning/active/docs-{slug}/`

**Spawn 3 sub-agents IN PARALLEL:**

### Agent 1: Requirements Writer

Dispatch `requirements-writer` agent with:
- Understanding summary from Step 5
- Codebase research from Step 3 (if brownfield)
- Confirmed tech stack from Step 2
- Output path: `.planning/active/docs-{slug}/requirements.md`

### Agent 2: Use Case Writer

Dispatch `usecase-writer` agent with:
- Understanding summary from Step 5
- Codebase research from Step 3 (if brownfield)
- Confirmed tech stack from Step 2
- Output path: `.planning/active/docs-{slug}/use-cases.md`

### Agent 3: Diagram Generator

Dispatch `diagram-generator` agent with:
- Understanding summary from Step 5
- Codebase research from Step 3 (if brownfield)
- Confirmed tech stack from Step 2
- Output path: `.planning/active/docs-{slug}/diagrams.md`
- Instruction: validate Mermaid syntax before finalizing

**Wait for all 3 agents to complete before proceeding.**

### Anti-Rationalization

| Excuse | Reality |
|--------|---------|
| "I can write all docs in one pass" | Parallel agents produce better results — each focuses on its domain without context pollution. |
| "Diagrams can be added later" | Diagrams generated alongside text are more consistent. Adding them later causes misalignment. |
| "Use cases are redundant with requirements" | Requirements say WHAT, use cases say HOW. Both are needed. |

---

## Step 7: Review & Compile

After all agents complete:

### 1. Cross-check outputs

- Requirements reference the same entities as use cases and diagrams
- Diagram labels match terminology used in requirements and use cases
- No contradictions between documents
- No placeholders left unresolved (search for `TBD`, `TODO`, `[NEEDS`)

Fix any inconsistencies inline. Do not ask the user to fix cross-agent mismatches.

### 2. Write analysis document

Write `.planning/active/docs-{slug}/analysis.md`:

```markdown
# Analysis: {Feature Name}

## Context
{What prompted this documentation, 2-3 sentences}

## Understanding Summary
{Copy from Step 5 — the confirmed version}

## Research Findings
{Summary of codebase/external research from Step 3}

## Approach
{How the feature should be implemented — high-level direction}

## Assumptions
1. {assumption}

## Open Questions
- {deferred to implementation phase}
```

### 3. Write index

Write `.planning/active/docs-{slug}/index.md`:

```markdown
# Feature Documentation: {Feature Name}

| Field   | Value          |
|---------|----------------|
| Status  | COMPLETE       |
| Date    | {YYYY-MM-DD}   |

## Summary
{1-2 sentences: what was documented and why}

## Documents
- [analysis.md](./analysis.md) — context, research, understanding
- [requirements.md](./requirements.md) — R001-R00N with acceptance criteria and estimates
- [use-cases.md](./use-cases.md) — actor-based flows
- [diagrams.md](./diagrams.md) — Mermaid diagrams

## Total Requirements
{N} requirements ({M} MUST, {K} SHOULD, {J} COULD)

## Estimated Effort
{sum of requirement estimate ranges}
```

### 4. Present to user

```
Documentation saved to `.planning/active/docs-{slug}/`.
- Requirements: {N} items (R001-R00{N})
- Use cases: {M} use cases with {K} flows
- Diagrams: {J} Mermaid diagrams
- Estimated effort: {range}

`approve` → commit docs | `reject` → tell me what to fix.
```

<HARD-GATE>
Wait for explicit approval before committing.
On reject → ask what to fix, update the specific document, re-present.
</HARD-GATE>

---

## Step 8: Commit

On approval:

```bash
git add .planning/active/docs-{slug}/
git commit -m "docs: add {feature-name} specification"
```

---

## Exit Criteria (ALL must be true)

- [ ] Project context read and understood (Step 1)
- [ ] Tech stack confirmed by user (Step 2)
- [ ] Research done — codebase and/or external (Step 3)
- [ ] Critical questions answered or documented as assumptions (Step 4)
- [ ] Understanding Lock confirmed by user (Step 5)
- [ ] All 3 agents completed and outputs cross-checked (Step 6-7)
- [ ] All artifacts written to `.planning/active/docs-{slug}/`
- [ ] User approved
- [ ] Committed to git

**If ANY criterion is unmet: do NOT commit.**

---

## Anti-Rationalization Table

| Excuse | Reality |
|--------|---------|
| "This feature is too simple for full docs" | Simple features with hidden complexity cause the most rework. Document anyway. |
| "Requirements are obvious" | Obvious to you ≠ obvious to the implementer. Write them down. |
| "Diagrams take too long" | Diagrams run in parallel — they add no time to the pipeline. |
| "I can skip the stack confirmation" | Wrong tech assumptions produce requirements that mislead implementation. |
| "One document is enough" | Requirements, use cases, and diagrams serve different audiences and purposes. |
| "I'll validate diagrams later" | Invalid Mermaid syntax compounds — validate immediately. |
