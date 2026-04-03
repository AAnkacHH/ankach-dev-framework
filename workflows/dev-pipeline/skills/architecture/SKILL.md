---
name: architecture
description: >
  Phase 2: Macro architecture decisions — servers, communication between systems,
  technologies, integrations, infrastructure. Use after Phase 1 Analysis is approved.
  MUST complete before Phase 3: Design.
---

# Phase 2: Architecture

<HARD-GATE>
This phase requires a completed and approved Phase 1 Analysis.
Read `.planning/active/01-analysis/index.md` — if it does not exist or Status is not APPROVED, stop and run Phase 1 first.
No implementation, no micro-design decisions until this phase is approved.
</HARD-GATE>

## What This Phase Covers (Macro — BETWEEN systems)

- Server / runtime decisions (new service? add to existing?)
- Inter-system communication (REST, message bus, webhooks, gRPC)
- Technology choices with rationale (new libraries, frameworks)
- External integrations (which APIs, which protocols)
- Infrastructure changes (queues, cache, new databases)
- Security boundaries (auth flow, credential handling)

## What This Phase Does NOT Cover (Micro — that's Phase 3)

- Entity fields, relations, migrations
- API endpoint contracts, DTOs, validation rules
- Internal service methods, repository design
- File-level responsibilities

---

## Step 1: Context (Silent)

Read and internalize:

1. `.planning/active/01-analysis/index.md` — chosen approach, requirements, risk assessment
2. `.planning/active/01-analysis/codebase-research.md` — existing patterns (if brownfield)
3. `.planning/active/01-analysis/external-research.md` — external findings (if exists)
4. `CLAUDE.md` + `AGENTS.md` — project rules (refresh if new session)
5. `.planning/STATE.md` — verify we're on Phase 2

**Do not show this step to the user.**

---

## Step 2: Gray Areas

Identify ambiguities in macro architecture that have multiple valid paths.

**For brownfield projects, check:**
- Does the chosen approach require new communication patterns? (e.g., new Wolverine message type)
- Are there external integrations not yet decided? (which carrier API, which protocol)
- Infrastructure changes needed? (new queue, cache layer, database)
- Security implications? (new auth flow, credential storage)

**For greenfield projects, check:**
- Server/runtime selection
- Database choice
- Communication patterns (REST, gRPC, message bus)
- Hosting/deployment target
- Authentication strategy

**Present gray areas to user as multiple-choice options:**

```
I found {N} architecture decisions that need your input:

**1. {Gray Area}**
   - Option A: {description} — {trade-off}
   - Option B: {description} — {trade-off}
   - My recommendation: {X} because {reason}

**2. {Gray Area}**
   ...
```

If the chosen approach from Phase 1 already resolves all macro decisions, state that explicitly and move to Step 4.

---

## Step 3: Questions

For any gray areas that need more context, ask targeted questions.

<RULES>
- ONE question at a time (same rules as Phase 1)
- Focus on MACRO decisions only — don't ask about entity fields or endpoint contracts
- If user defers a decision, document it as "deferred to Phase 3" with your recommended default
</RULES>

**"3 Questions" gate — ask before adopting any pattern or technology:**

1. What SPECIFIC problem does this solve?
2. Is there a simpler alternative?
3. Can we add this LATER when needed?

If the answer to #3 is "yes" — defer it. YAGNI.

---

## Step 4: Architecture Decisions

For each macro decision made (from gray areas + questions), write an ADR (Architecture Decision Record).

**ADR format:**

```markdown
### ADR-{N}: {Decision Title}

**Status:** Accepted
**Date:** {YYYY-MM-DD}

**Context:**
{What problem are we solving? What constraints exist?}

**Options Considered:**

| Option | Pros | Cons | Complexity |
|--------|------|------|------------|
| {A} | {benefit} | {drawback} | Low/Med/High |
| {B} | {benefit} | {drawback} | Low/Med/High |

**Decision:** {What we chose}

**Rationale:** {Why — tied to requirements and constraints from Phase 1}

**Consequences:**
- Positive: {benefits we gain}
- Negative: {costs we accept}
- Mitigation: {how we address negatives}
```

**For brownfield projects** where most architecture is already decided:
- Write a brief "Architecture Confirmation" instead of full ADRs
- Only write full ADRs for NEW decisions
- Reference existing patterns: "Following existing pattern from {X}"

---

## Step 5: System Design Checklist

Verify nothing is missed across 4 categories:

**Functional:**
- [ ] All Phase 1 requirements can be implemented with this architecture
- [ ] Data flow is clear (where data comes from, where it goes)
- [ ] Error scenarios have a handling strategy

**Non-Functional:**
- [ ] Performance: no obvious bottlenecks in the architecture
- [ ] Security: auth, credential handling, data protection addressed
- [ ] Scalability: architecture can handle expected load

**Technical:**
- [ ] Technology choices are compatible with existing stack
- [ ] No deprecated or EOL technologies chosen
- [ ] Integration points are well-defined

**Operations:**
- [ ] Database migrations strategy identified (if schema changes)
- [ ] Configuration/environment variables identified
- [ ] Monitoring/logging approach clear

Mark each as PASS / FAIL / N/A. If any FAIL — address before proceeding.

---

## Step 6: Approval + Document

### 1. Write `.planning/active/02-architecture/index.md`:

```markdown
# Phase 2: Architecture — {Feature Name}

| Field    | Value                    |
|----------|--------------------------|
| Phase    | 2 of 9                  |
| Status   | APPROVED                |
| Date     | {YYYY-MM-DD}            |

## Summary
{2-3 sentences: what architecture decisions were made}

## Architecture Overview
{How the feature fits into the system — communication flow, affected services}

## System Design Checklist
| Category | Status | Notes |
|----------|--------|-------|
| Functional | PASS/FAIL | {details} |
| Non-Functional | PASS/FAIL | {details} |
| Technical | PASS/FAIL | {details} |
| Operations | PASS/FAIL | {details} |

## Files in This Phase
- [tech-decisions.md](./tech-decisions.md) — ADRs
- [integrations.md](./integrations.md) — external services
- [infrastructure.md](./infrastructure.md) — infra changes (if any)

## Gate
Approved → Phase 3: Design
```

### 2. Write `.planning/active/02-architecture/tech-decisions.md`:

All ADRs from Step 4 (or "Architecture Confirmation" for brownfield with no new decisions).

### 3. Write `.planning/active/02-architecture/integrations.md` (if external integrations):

```markdown
# Integrations: {Feature Name}

## External Services
| Service | Protocol | Direction | Auth | Notes |
|---------|----------|-----------|------|-------|
| {Carrier API} | REST | Outbound | OAuth2 | {details} |

## Data Flow
{How data flows between systems}
```

### 4. Write `.planning/active/02-architecture/infrastructure.md` (if infra changes):

```markdown
# Infrastructure Changes: {Feature Name}

## New Components
- {component}: {purpose}

## Configuration
- {env var}: {purpose}

## Migration Notes
- {what needs to change in infrastructure}
```

### 5. Present to user:

```
Architecture documented in `.planning/active/02-architecture/`.
{N} ADRs written. System design checklist: all PASS.

Please review. `approve` → Phase 3: Design | `reject` → tell me what to fix.
```

<HARD-GATE>
Wait for explicit approval before Phase 3.
On approve → update .planning/STATE.md (Phase 2 completed, Phase 3 pending).
On reject → fix and re-present.
</HARD-GATE>

---

## Exit Criteria (ALL must be true)

- [ ] Phase 1 analysis read and understood (Step 1)
- [ ] Gray areas identified and resolved with user (Steps 2-3)
- [ ] "3 Questions" gate applied to every new pattern/technology
- [ ] ADR written for every macro decision (Step 4)
- [ ] System Design Checklist passed (Step 5)
- [ ] All artifacts written to `.planning/active/02-architecture/` (Step 6)
- [ ] User approved
- [ ] STATE.md updated

---

## Anti-Rationalization Table

| Excuse | Reality |
|--------|---------|
| "Architecture is obvious for this feature" | Document it anyway. Obvious ≠ agreed upon. |
| "We already use this pattern" | Confirm it applies here. Existing pattern ≠ always correct. |
| "Let's just start coding and figure it out" | Architecture decisions made during coding are never revisited. Decide now. |
| "This is just a small feature, no architecture needed" | Small features that touch external systems need architecture review. |
| "The tech stack is already decided" | New features may need new integrations, queues, or protocols. Check. |
