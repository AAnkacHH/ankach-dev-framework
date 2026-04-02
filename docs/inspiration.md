# Inspiration

This framework was built after deep analysis of 4 major Claude Code frameworks. Each was researched by dedicated agents that read actual source files, not just READMEs.

## Frameworks Analyzed

| Framework | Stars | Description |
|-----------|-------|-------------|
| [**Superpowers**](https://github.com/obra/superpowers) | 40k+ | Skills-based workflow by Jesse Vincent. 14 skills, strict TDD, two-stage review. |
| [**GSD**](https://github.com/gsd-build/get-shit-done) | 43k+ | Meta-prompting system. 56 workflows, 18 agents, wave-based execution. |
| [**Everything Claude Code**](https://github.com/affaan-m/everything-claude-code) | 50k+ | Multi-platform plugin. 125 skills, 28 agents, continuous learning pipeline. |
| [**Antigravity**](https://github.com/sickn33/antigravity-awesome-skills) | 28k+ | Skills marketplace. 1,329 skills, state machines, risk classification. |

## What We Took

### From Superpowers
- **Anti-rationalization tables** — excuses Claude generates and why they're wrong (12-row tables proven to reduce step-skipping)
- **`<HARD-GATE>` XML tags** — LLM responds to XML tags stronger than bold text
- **"Do not trust the report"** — reviewer independently reads code, never trusts implementer claims
- **One question per message** — prevents overwhelming the user
- **Spec self-review** — 4 checks before presenting to user (placeholders, contradictions, scope, ambiguity)
- **Description = trigger only, not summary** — prevents Claude from following abbreviated instructions

### From GSD
- **`.planning/` as shared state** — file-based inter-phase communication that survives session resets
- **Wave-based parallel execution** — dependency-aware grouping for parallel agent dispatch
- **4-level deviation rules** — auto-fix for bugs/missing/blockers, STOP for architectural decisions
- **Gray areas identification** — find ambiguities in decisions, present as multiple-choice
- **Goal-backward verification** — check "is goal achieved" not "are tasks done"
- **Atomic commits per phase** — if session is lost, artifacts persist
- **Analysis paralysis guard** — 5+ reads without writes = STOP and report blocker

### From Everything Claude Code
- **Tool whitelist per agent** — READ-ONLY reviewers (`tools: Read, Grep, Glob` — no Write, no Edit)
- **Model routing** — Opus for quality review (stronger reasoning), Sonnet for implementation (faster)
- **Structured verdicts** — SHIP / NEEDS WORK / BLOCKED (not free-form text)
- **Handoff documents** — structured format between agents (Context, Findings, Files, Questions, Recommendations)
- **Architect vs Planner separation** — "what and why" (read + shell) vs "how and when" (read only)
- **System Design Checklist** — functional, non-functional, technical, operations

### From Antigravity
- **Understanding Lock** — 5-7 bullet summary → explicit user confirmation before any design work
- **Decision Log** — mandatory throughout, captures what/alternatives/why for every decision
- **Exit criteria as hard stops** — ALL criteria must be true, not "most"
- **"3 Questions" gate** — before any pattern: what problem? simpler alternative? can we defer?
- **Risk-based escalation** — low=proceed, moderate=recommend review, high=require review
- **State machine patterns** — formal states with transition rules and evidence requirements

## Research Archive

8 detailed comparison documents are preserved in `.planning/research/`:

| File | Content |
|------|---------|
| `phase1-analysis-comparison.md` | How each framework handles analysis/brainstorming |
| `phase1-agent-proposal.md` | Phase 1 agent design (v2, 6-step workflow) |
| `phase2-3-architecture-comparison.md` | Architecture and design patterns across frameworks |
| `phase4-decomposition-comparison.md` | Task decomposition approaches vs existing structure |
| `phase5-validation-comparison.md` | Pre-implementation validation gates |
| `phase6-implementation-comparison.md` | Parallel execution, dispatch, deviation rules |
| `phase7-review-comparison.md` | Code review stages, verdicts, feedback loops |
| `phase8-9-docs-deploy-comparison.md` | Documentation generation and deployment strategies |
