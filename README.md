<p align="center">
  <h1 align="center">Ankach Dev Framework</h1>
  <p align="center">
    A 9-phase AI-assisted development pipeline for <a href="https://claude.ai/code">Claude Code</a>
    <br />
    <em>Skills, agents, and structured workflows for building software the right way</em>
  </p>
  <p align="center">
    <a href="#quick-start">Quick Start</a> &middot;
    <a href="#workflows">Workflows</a> &middot;
    <a href="#9-phases">9 Phases</a> &middot;
    <a href="#agents">Agents</a> &middot;
    <a href="#installation">Installation</a>
  </p>
</p>

---

## Why?

AI coding assistants are powerful but chaotic. They skip analysis, jump to code, miss edge cases, and produce inconsistent results. This framework adds structure:

- **Every feature goes through 9 phases** with documented artifacts
- **You control every step** via manual approval gates
- **Agents can't cut corners** thanks to anti-rationalization tables and hard gates
- **Nothing is lost** because every decision, approach, and trade-off is documented

Built by combining the best patterns from [Superpowers](https://github.com/obra/superpowers), [GSD](https://github.com/gsd-build/get-shit-done), [Everything Claude Code](https://github.com/affaan-m/everything-claude-code), and [Antigravity Awesome Skills](https://github.com/sickn33/antigravity-awesome-skills).

## Quick Start

```bash
# Clone the framework
git clone https://github.com/AAnkacHH/ankach-dev-framework.git

# Copy into your project
cp -r ankach-dev-framework/.claude/ your-project/.claude/

# Start building
cd your-project
# For a new feature:
# /feature Add user authentication with OAuth
# For a new project:
# /workflow-full Build a REST API for inventory management
```

## Workflows

Two entry points depending on your use case:

| Command | Use Case | Skippable Phases |
|---------|----------|-----------------|
| `/feature <description>` | Add feature to **existing project** | Phase 2 (Architecture), Phase 8 (Docs) |
| `/workflow-full <description>` | **New project** or major feature | None |

Both walk through all 9 phases sequentially. Each phase reads the full skill definition and executes every step. You approve at every gate.

```
/feature Add bulk shipment endpoint

  Phase 1: Analysis .............. approve
  Phase 2: Architecture .......... skip (no macro changes)
  Phase 3: Design ................ approve
  Phase 4: Decomposition ......... approve
  Phase 5: Validation ............ PASS
  Phase 6: Implementation ........ approve (wave by wave)
  Phase 7: Review ................ SHIP
  Phase 8: Documentation ......... approve
  Phase 9: Deploy ................ done -> archived
```

You can also run phases individually: `/analyze`, `/architect`, `/design`, etc.

## Usage Examples

### Example 1: Add a feature to existing project

```
> /feature Add bulk shipment endpoint that accepts up to 100 shipments

Phase 1: Analysis
─────────────────
I've read the codebase. ShipmentController exists, Wolverine is used for async.

Understanding Summary:
- What: POST /api/v1/shipments/bulk accepting up to 100 shipments
- Why: Clients need to create multiple shipments in one API call
- Who: API consumers (e-commerce platforms)
- Scope: New endpoint + service + validation
- Not included: UI changes, carrier-specific batching
- Constraints: Rate limiting, max 100 per request

> Does this accurately reflect your intent?

You: approve

Phase 2: Architecture
─────────────────────
No new macro architecture decisions needed.
Existing stack applies: Wolverine for async, PostgreSQL, existing carrier adapters.

You: skip

Phase 3: Design
───────────────
Domain Map: Shipment subdomain (Major), Carrier (Minor), Organization (None)

New entity: BulkShipmentRequest (id, status, items_count, ...)
New endpoint: POST /api/v1/shipments/bulk
  Request: { shipments: [...] }
  Response: { success: true, data: { request_id, results: [...] } }

You: approve

Phase 4: Decomposition
──────────────────────
Epic: Bulk Shipment

Story 1: BulkShipment entity + migration     | Wave A
Story 2: BulkShipmentService                  | Wave B ← (Story 1)
Story 3: BulkShipmentController + validation  | Wave B ← (Story 1)
Story 4: Wolverine async handler              | Wave C ← (Story 2, 3)

You: approve

Phase 5: Validation ......... PASS
Phase 6: Implementation ..... 4 stories across 3 waves
Phase 7: Review ............. SHIP (0 critical, 0 high)
Phase 8: Documentation ...... 1 endpoint documented
Phase 9: Deploy ............. PR created, archived

✓ Feature complete.
```

### Example 2: New project from scratch

```
> /workflow-full Build a webhook relay service that receives carrier webhooks
  and forwards them to customer endpoints with retry logic

Phase 1: Analysis
─────────────────
This is a greenfield project. Let me research the domain...
[spawns researcher agent for stack/patterns/pitfalls]

Approaches:
A: Minimal — Express + PostgreSQL + simple retry queue
B: Event-driven — Fastify + Redis Streams + dead letter queue
C: Serverless — AWS Lambda + SQS + DynamoDB

My recommendation: B — best balance of reliability and simplicity.

You: go with B

Phase 2: Architecture
─────────────────────
ADR-1: Use Fastify over Express (faster, schema validation built-in)
ADR-2: Use Redis Streams for webhook queue (reliable, ordered, consumer groups)
ADR-3: Use PostgreSQL for webhook logs and customer configs

You: approve

Phase 3-9: [continues through all phases, no skipping]
```

### Example 3: Run a single phase

```
> /analyze Refactor the credential encryption to use AES-256-GCM instead of AES-256-CBC

[runs only Phase 1 — analysis, research, questions, approaches]
[does NOT auto-continue to Phase 2]

Analysis saved to .planning/active/01-analysis/.
Chosen approach: In-place migration with backward compatibility.

> approve

To continue: /architect
```

### Example 4: Resume interrupted workflow

```
> /feature continue

Reading STATE.md...
Feature: Bulk Shipment Endpoint
Last completed: Phase 4 (Decomposition)
Resuming from: Phase 5 (Validation)

Phase 5: Validation
───────────────────
[continues from where you left off]
```

### User commands during workflow

| Command | What it does |
|---------|-------------|
| `approve` | Accept phase, move to next |
| `reject` | Stay in phase, fix issues |
| `skip` | Skip phase (only Phase 2, 8) |
| `pause` | Save state, resume later with `/feature continue` |
| `abort` | Stop workflow, keep all artifacts |

## 9 Phases

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

<details>
<summary><b>Phase details (click to expand)</b></summary>

### Phase 1: Analysis (`/analyze`)
- Read project context silently (CLAUDE.md, AGENTS.md, .context/)
- Detect brownfield/greenfield automatically
- Research codebase or external sources
- Ask clarifying questions (one at a time, max 7)
- **Understanding Lock** — confirm understanding before proposing anything
- Propose 2-3 approaches with trade-offs
- **Output:** `.planning/active/01-analysis/`

### Phase 2: Architecture (`/architect`)
- Identify gray areas in macro decisions
- Apply "3 Questions" gate: What problem? Simpler alternative? Can we defer?
- Write ADR for each technology decision
- System design checklist (functional, non-functional, technical, operations)
- **Output:** `.planning/active/02-architecture/`

### Phase 3: Design (`/design`)
- Domain map with subdomains, bounded contexts, impact assessment
- Data model: entities, fields, relations, migrations
- API contracts: endpoints, DTOs, validation, status codes
- Self-review: coverage, consistency, no placeholders
- **Output:** `.planning/active/03-design/`

### Phase 4: Decomposition (`/decompose`)
- Epic definition with goal and acceptance criteria
- Stories with `Depends on:` and `Wave:` fields
- Sub-tasks with exact file paths (designed for AI agent delegation)
- Wave plan for parallel execution
- **Output:** `.planning/active/04-decomposition/`

### Phase 5: Validation (`/validate`)
- **Architecture review:** deprecated tech, anti-patterns, security, performance, framework misuse
- Requirements coverage: every requirement maps to a story
- Story completeness: AC, dependencies, waves, sub-tasks
- PASS/FAIL verdict. Max 2 iterations before escalation
- **Output:** `.planning/active/05-validation/`

### Phase 6: Implementation (`/build`)
- Create feature branch, verify baseline tests
- Execute waves: parallel agents per wave, sequential waves
- Deviation rules: auto-fix bugs (R1-3), STOP for architecture (R4)
- Agent status: DONE / DONE_WITH_CONCERNS / BLOCKED / NEEDS_CONTEXT
- **Output:** `.planning/active/06-implementation/`

### Phase 7: Review (`/review`)
- **Staged (fail-fast):** spec compliance BEFORE code quality
- Spec compliance: read CODE, not agent reports ("do not trust the report")
- Code quality: SOLID, security, performance, conventions
- Test verification: all pass, coverage, edge cases
- Verdict: SHIP / NEEDS WORK / BLOCKED
- **Output:** `.planning/active/07-review/`

### Phase 8: Documentation (`/document`)
- Generate from CODE (source of truth), not design docs
- API docs for new/changed endpoints
- Dev notes: how it works, gotchas
- Changelog entry (Keep a Changelog format)
- **Output:** `.planning/active/08-documentation/`

### Phase 9: Deploy (`/deploy`)
- Pre-deploy checklist (tests, review, secrets, migrations, docs)
- Migration plan with rollback strategy
- Create PR or merge locally
- Post-deploy verification (health check, smoke tests)
- Archive `.planning/active/` → `docs/features/YYYY-MM-DD-{slug}/`
- **Output:** `.planning/active/09-deploy/`

</details>

## Agents

Five specialized agents with scoped tool access:

| Agent | Phase | Tools | Model | Role |
|-------|-------|-------|-------|------|
| `researcher` | 1, 2 | Read, Write, Grep, Glob, Bash, WebSearch, WebFetch | Sonnet | Codebase + external research |
| `implementer` | 6 | Read, Write, Edit, Grep, Glob, Bash | Sonnet | Implement a single story |
| `test-writer` | 6 | Read, Write, Edit, Grep, Glob, Bash | Sonnet | Write unit + integration tests |
| `spec-reviewer` | 7 | Read, Grep, Glob | Sonnet | Spec compliance (READ-ONLY) |
| `code-quality-reviewer` | 7 | Read, Grep, Glob, Bash | Opus | Code quality review (READ-ONLY) |

> Reviewers are deliberately **read-only** — they cannot modify code, only report findings.

## Key Patterns

### Hard Gates
Every phase ends with a `<HARD-GATE>` — Claude cannot proceed without your explicit `approve`.

### Anti-Rationalization Tables
Every skill includes a table of excuses Claude might generate and why they're wrong:

| Excuse | Reality |
|--------|---------|
| "This is too simple for analysis" | Simple tasks have hidden assumptions. 5 min of analysis prevents hours of rework. |
| "The agent report says everything is fine" | Do NOT trust the report. Read the code yourself. |
| "Tests pass, so the code is correct" | Tests passing ≠ spec compliance. Check requirements. |

### Deviation Rules (Phase 6)

| Rule | Trigger | Action |
|------|---------|--------|
| Rule 1 | Bug — code doesn't work | Auto-fix (max 3 attempts) |
| Rule 2 | Missing validation/auth/error handling | Auto-add (max 3 attempts) |
| Rule 3 | Missing deps, broken imports | Auto-resolve (max 3 attempts) |
| Rule 4 | Architectural decision needed | **STOP — ask user** |

### Understanding Lock (Phase 1)
Before proposing any approach, Claude must present a summary of what it understands and get explicit confirmation. Prevents building solutions for the wrong problem.

### Validation Loops
- **Phase 5 → Phase 3/4:** If requirements don't match decomposition
- **Phase 7 → Phase 4/5/6:** If code doesn't match spec or has quality issues

Max 2 iterations per loop before escalating to the user.

## Installation

### Option 1: Copy into your project

```bash
git clone https://github.com/AAnkacHH/ankach-dev-framework.git
cp -r ankach-dev-framework/.claude/skills/ your-project/.claude/skills/
cp -r ankach-dev-framework/.claude/agents/ your-project/.claude/agents/
cp -r ankach-dev-framework/.claude/commands/ your-project/.claude/commands/
```

### Option 2: Add as git submodule

```bash
git submodule add https://github.com/AAnkacHH/ankach-dev-framework.git .framework
# Then symlink or copy what you need
```

### Configure CLAUDE.md

Add to your project's `CLAUDE.md`:

```markdown
## Development Framework
- Skills: `.claude/skills/` — MANDATORY when triggered
- Agents: `.claude/agents/` — scoped tool access per role
- Hard gates between phases (no skipping)
- Subagents get ONLY task-specific context
- Use `/feature <description>` for new features
- Use `/workflow-full <description>` for new projects
```

## File Structure

```
.claude/
├── commands/                          # Slash commands (thin triggers)
│   ├── analyze.md                     # /analyze — Phase 1
│   ├── architect.md                   # /architect — Phase 2
│   ├── design.md                      # /design — Phase 3
│   ├── decompose.md                   # /decompose — Phase 4
│   ├── validate.md                    # /validate — Phase 5
│   ├── build.md                       # /build — Phase 6
│   ├── review.md                      # /review — Phase 7
│   ├── document.md                    # /document — Phase 8
│   ├── deploy.md                      # /deploy — Phase 9
│   ├── feature.md                     # /feature — brownfield workflow
│   ├── workflow-full.md               # /workflow-full — full pipeline
│   └── map-codebase.md               # /map-codebase — utility
│
├── skills/                            # Full workflow definitions
│   ├── analysis/SKILL.md
│   ├── architecture/SKILL.md
│   ├── design/SKILL.md
│   ├── decomposition/SKILL.md
│   ├── validation/SKILL.md
│   ├── implementation/SKILL.md
│   ├── review/SKILL.md
│   ├── documentation/SKILL.md
│   ├── deploy/SKILL.md
│   ├── workflow-full/SKILL.md         # Orchestrator: all 9 phases
│   └── workflow-feature/SKILL.md      # Orchestrator: brownfield
│
└── agents/                            # Sub-agent definitions
    ├── researcher.md
    ├── implementer.md
    ├── test-writer.md
    ├── spec-reviewer.md
    └── code-quality-reviewer.md

.planning/                             # Generated at runtime
├── STATE.md                           # Current feature state
├── active/                            # Phase artifacts (01-analysis/ ... 09-deploy/)
└── research/                          # Framework research archive
```

## Inspired By

This framework was built after deep analysis of 4 major Claude Code frameworks. Each was researched by dedicated agents that read actual source files (not just READMEs):

| Framework | Stars | What We Took |
|-----------|-------|-------------|
| [**Superpowers**](https://github.com/obra/superpowers) | 40k+ | Anti-rationalization tables, `<HARD-GATE>` XML tags, "do not trust the report" reviewer pattern, one question per message |
| [**GSD**](https://github.com/gsd-build/get-shit-done) | 43k+ | `.planning/` as shared state, wave-based parallel execution, 4-level deviation rules, goal-backward verification |
| [**Everything Claude Code**](https://github.com/affaan-m/everything-claude-code) | 50k+ | Tool whitelist per agent (READ-ONLY reviewers), model routing (Opus for review, Sonnet for implementation), structured verdicts |
| [**Antigravity**](https://github.com/sickn33/antigravity-awesome-skills) | 28k+ | Understanding Lock, mandatory Decision Log, exit criteria as hard stops, "3 Questions" gate before any pattern |

8 detailed comparison documents are preserved in `.planning/research/`.

## License

MIT &copy; 2026 [Andrii Plyskach](https://github.com/AAnkacHH)
