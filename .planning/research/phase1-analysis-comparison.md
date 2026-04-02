# Phase 1: Analysis — Comparison of 4 Frameworks

**Date:** 2026-03-28
**Purpose:** What skills, agents, rules, and patterns each framework uses for the Analysis/Brainstorming phase

---

## 1. Superpowers

### Skills Used
| Skill | Purpose |
|-------|---------|
| `brainstorming/SKILL.md` | Main analysis — 9-step checklist |
| `using-superpowers/SKILL.md` | Meta-skill (always loaded, forces skill invocation) |

### Agents
None for analysis. Spec-document-reviewer is a prompt template inside the skill dir, dispatched as a generic subagent.

### Rules / Enforcement
- `<HARD-GATE>` XML tags — LLM responds stronger than bold text
- Anti-rationalization table (12 rows: "This is just a simple question" → "Questions are tasks. Check for skills.")
- Anti-pattern section: "This is too simple to need a design" — explicitly addressed
- Description = trigger only, NEVER summary (tested: summaries cause Claude to skip steps)

### Process
1. Explore project context (files, docs, commits)
2. Offer visual companion (optional)
3. Ask clarifying questions — **one at a time**, prefer multiple choice
4. Propose 2-3 approaches with trade-offs + recommendation
5. Present design incrementally (200-300 word sections, confirm after each)
6. Write design doc → `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`
7. Spec self-review (placeholder scan, consistency, scope, ambiguity)
8. **User reviews written spec** — HARD GATE
9. Transition → writing-plans ONLY

### Key Patterns to Steal
- **One question per message** — never overwhelm
- **Spec self-review** before user review (4 checks: placeholders, contradictions, scope, ambiguity)
- **Terminal state lock** — "The ONLY skill you invoke after brainstorming is writing-plans"
- **Anti-rationalization tables** — proven effective against Claude skipping steps
- **Description ≠ summary** — critical for correct skill triggering

---

## 2. GSD (Get Shit Done)

### Skills / Workflows Used
| File | Purpose |
|------|---------|
| `workflows/new-project.md` | 39KB orchestrator — full init flow |
| `references/questioning.md` | Questioning philosophy + patterns |
| `references/checkpoints.md` | 3 checkpoint types |
| `templates/project.md` | PROJECT.md template |
| `templates/requirements.md` | Requirements template |
| `templates/state.md` | STATE.md template |
| `templates/config.json` | Default config with gates |

### Agents Spawned During Analysis
| Agent | Count | Purpose | Tools |
|-------|-------|---------|-------|
| `gsd-codebase-mapper` | x4 parallel | tech, arch, quality, concerns | Read, Bash, Grep, Glob, Write |
| `gsd-project-researcher` | x4 parallel | Stack, Features, Architecture, Pitfalls | Read, Write, Bash, Grep, Glob, WebSearch, WebFetch |
| `gsd-research-synthesizer` | x1 | Merge 4 research outputs → SUMMARY.md | Read, Write, Bash |
| `gsd-roadmapper` | x1 | Requirements → Phases → ROADMAP.md | Read, Write, Bash, Glob, Grep |
| `gsd-assumptions-analyzer` | x1 | Phase assumptions with evidence | Read, Bash, Grep, Glob |

### Rules / Enforcement
- **Questioning guide**: "Dream extraction, not requirements gathering"
- **Context checklist**: What/Why/Who/Done (just 4 things)
- **Config-driven gates**: `confirm_project`, `confirm_phases`, `confirm_roadmap`
- **CLI tool for deterministic logic** — `gsd-tools.cjs` for brownfield detection, not LLM

### Process
1. CLI setup (detect git, brownfield, existing maps)
2. Brownfield gate → offer `/gsd:map-codebase` (4 parallel mapper agents)
3. Deep questioning (follow threads, challenge vagueness, 4-item checklist)
4. Write PROJECT.md → commit
5. Workflow preferences (mode, granularity, agents, models) → config.json
6. Research decision → 4 parallel researchers + 1 synthesizer
7. Define requirements (category-by-category scoping, REQ-ID format)
8. Create roadmap → roadmapper agent → ROADMAP.md + STATE.md

### Key Patterns to Steal
- **File-based inter-agent communication** — `.planning/` as shared state
- **4 parallel research agents** — Stack, Features, Architecture, Pitfalls
- **Deep questioning philosophy**: follow energy, challenge vagueness, don't interrogate
- **Brownfield detection** — if existing code, map it first
- **REQ-ID format**: `[CATEGORY]-[NUMBER]` (AUTH-01, API-02)
- **Config.json** for gate control (which gates are active, auto/interactive mode)
- **Tool confidence levels**: HIGH (official docs), MEDIUM (verified search), LOW (unverified)
- **Atomic commits per phase** — if context lost, artifacts persist

---

## 3. Everything Claude Code (ECC)

### Skills Used
| Skill | Purpose |
|-------|---------|
| `blueprint` | Multi-session planning (5 phases) |
| `deep-research` | Web research (15-30 sources) |
| `search-first` | Research before coding |
| `codebase-onboarding` | 4-phase onboarding for unfamiliar codebases |
| `strategic-compact` | Context window management |
| `verification-loop` | 6-phase post-implementation check |

### Agents
| Agent | Tools | Model | Purpose |
|-------|-------|-------|---------|
| `planner` | Read, Grep, Glob | **opus** | READ-ONLY planning |
| `architect` | Read, Grep, Glob | **opus** | READ-ONLY architecture |

### Rules
| Rule File | Key Content |
|-----------|-------------|
| `rules/common/coding-style.md` | Immutability, file org (<800 lines), quality checklist |
| `rules/common/patterns.md` | Repository pattern, API response format |
| `rules/common/security.md` | Mandatory pre-commit checklist |
| `rules/common/development-workflow.md` | Phase 0: Research & Reuse |
| `rules/common/agents.md` | When to auto-invoke which agent |
| `rules/common/performance.md` | Model routing: Haiku/Sonnet/Opus |

### Contexts (Mode Switching)
| Context | Behavior |
|---------|----------|
| `dev.md` | "Write code first, explain after" — favor Edit, Write, Bash |
| `research.md` | "Read widely before concluding" — favor Read, Grep, WebSearch |
| `review.md` | "Read thoroughly before commenting" — severity-first output |

### Process
1. Phase 0: Research & Reuse — search GitHub, vendor docs, package registries for 80%+ solutions
2. Planner agent (opus, read-only): requirements analysis → architecture review → step breakdown → implementation order
3. Plan format with phases, dependencies, risks, testing strategy
4. **Mandatory confirmation gate**: "The planner agent will NOT write any code until you explicitly confirm"
5. Handoff document → next agent in chain

### Key Patterns to Steal
- **Planning agents are READ-ONLY** — `tools: ["Read", "Grep", "Glob"]` prevents code changes
- **Model routing**: opus for planning/architecture, sonnet for implementation
- **Worked example in agent definition** — Stripe Subscriptions example shows expected detail level
- **Handoff documents** — structured format: Context, Findings, Files Modified, Open Questions, Recommendations
- **Phase 0: Research & Reuse** — always check if solution already exists
- **Context switching** — dev/research/review modes change agent behavior
- **Rules layering**: common/ (universal) + language-specific/ (override)
- **Structured verdicts**: SHIP / NEEDS WORK / BLOCKED

---

## 4. Antigravity Awesome Skills

### Skills Used
| Skill | Purpose |
|-------|---------|
| `brainstorming` | Understanding Lock + Decision Log |
| `concise-planning` | Quick atomic checklists |
| `writing-plans` | Detailed implementation plans |
| `architecture` | ADR framework with sub-files |
| `architect-review` | Architecture review methodology |
| `design-orchestration` | Routing between skills based on risk |
| `multi-agent-brainstorming` | Adversarial review (5 roles) |
| `acceptance-orchestrator` | 7-state machine |
| `systematic-debugging` | Iron Law + 4 phases |
| `verification-before-completion` | Evidence-based claims |
| `create-issue-gate` | Issue quality gate |
| `analyze-project` | Forensic 10-step project analysis |
| `closed-loop-delivery` | DoD verification |

### Agents
No standalone agents. Skills define behavior inline. Multi-agent brainstorming uses 5 sequential roles:
1. **Primary Designer** — proposes, revises
2. **Skeptic/Challenger** — "Assume this fails in production. Why?"
3. **Constraint Guardian** — performance, security, cost
4. **User Advocate** — cognitive load, usability
5. **Integrator/Arbiter** — resolves conflicts, declares completion

### Rules / Enforcement
- **Risk classification**: none / safe / critical / offensive (per skill)
- **Understanding Lock** — HARD GATE before any design
- **Iron Law** patterns (debugging, verification)
- **Common Rationalizations tables** (excuse → reality)
- **Exit criteria** — hard stop conditions, ALL must be true

### Process (via design-orchestration routing)
1. Brainstorming (mandatory) → Understanding Lock
2. Risk assessment → classify as low/moderate/high
3. Low risk → proceed to implementation planning
4. Moderate risk → recommend multi-agent review
5. High risk → REQUIRE multi-agent review (5 roles, sequential)
6. Execution readiness check → all criteria must pass

### Key Patterns to Steal
- **Understanding Lock** — 5-7 bullet summary (what/why/who/constraints/non-goals) → explicit confirmation
- **Decision Log** — mandatory throughout, captures what/alternatives/why for every decision
- **Risk-based escalation** — low=proceed, moderate=recommend review, high=require review
- **Multi-agent adversarial review** — 5 sequential roles with enforced perspectives
- **design-orchestration** — routing skill that enforces correct phase order
- **State machine** — formal states with transition rules and evidence requirements
- **create-issue-gate** — no acceptance criteria = issue stays `draft`, execution blocked
- **Exit criteria as hard stops** — ALL conditions must be true, not "most"

---

## Synthesis: Best Patterns for Ankach Analysis Agent

### From Superpowers
- [ ] Anti-rationalization tables in every skill
- [ ] `<HARD-GATE>` XML tags for enforcement
- [ ] One question per message rule
- [ ] Spec self-review (4 checks) before user review
- [ ] Description = trigger only, not summary

### From GSD
- [ ] File-based state (`.planning/` directory)
- [ ] Parallel research agents (codebase + external)
- [ ] Deep questioning philosophy (follow energy, challenge vagueness)
- [ ] Context checklist (what/why/who/done)
- [ ] Brownfield detection for existing codebases
- [ ] Atomic commits per phase artifact
- [ ] Config-driven gates

### From ECC
- [ ] Planning agents are READ-ONLY (tools whitelist)
- [ ] Model routing (opus for analysis, sonnet for implementation)
- [ ] Worked examples showing expected output quality
- [ ] Handoff documents between phases
- [ ] Phase 0: Research & Reuse
- [ ] Context switching (research mode for analysis)

### From Antigravity
- [ ] Understanding Lock (summary → confirm before design)
- [ ] Decision Log (mandatory throughout)
- [ ] Risk-based escalation (low/moderate/high → different review depth)
- [ ] Exit criteria as hard stops (ALL must be true)
- [ ] Multi-agent adversarial review for high-risk designs
- [ ] State machine patterns for complex workflows
