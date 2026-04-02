# Ankach Dev Framework

A 9-phase development pipeline for [Claude Code](https://claude.ai/code), combining the best patterns from Superpowers, GSD, Everything Claude Code, and Antigravity Awesome Skills.

## What Is This?

A structured workflow framework that guides AI-assisted development through 9 sequential phases with manual approval gates at every step. Works for both new projects (greenfield) and adding features to existing projects (brownfield).

## Pipeline

```
/analyze --> /architect --> /design --> /decompose --> /validate --> /build --> /review --> /document --> /deploy
```

## 9 Phases

| # | Phase | Command | Purpose |
|---|-------|---------|---------|
| 1 | Analysis | `/analyze` | Understand the task, research, propose 2-3 approaches |
| 2 | Architecture | `/architect` | Macro decisions: servers, communication, technologies |
| 3 | Design | `/design` | Micro decisions: domain map, entities, API contracts |
| 4 | Decomposition | `/decompose` | Break into EPIC -> STORIES -> TASKS with wave plan |
| 5 | Validation | `/validate` | Architecture review + requirements coverage check |
| 6 | Implementation | `/build` | Parallel agent execution, wave-by-wave |
| 7 | Review | `/review` | Spec compliance -> Code quality -> Tests |
| 8 | Documentation | `/document` | API docs, dev notes, changelog |
| 9 | Deploy | `/deploy` | Pre-deploy checklist, merge/PR, post-deploy verification |

## Key Features

- **Manual gates at every phase** -- you always control what happens
- **Two validation loops** -- Phase 5 can loop back to 3/4, Phase 7 can loop back to 4/5/6
- **Wave-based parallel implementation** -- independent stories run in parallel
- **Deviation rules** -- agents auto-fix bugs (Rules 1-3), but STOP for architectural decisions (Rule 4)
- **"Do not trust the report"** -- reviewers read actual code, not agent claims
- **Anti-rationalization tables** -- prevent Claude from skipping steps
- **Tech-agnostic** -- stack specifics live in your CLAUDE.md, skills work with any stack

## 5 Agents

| Agent | Purpose | Model |
|-------|---------|-------|
| `researcher` | Codebase + external research | sonnet |
| `implementer` | Implement a single story | sonnet |
| `test-writer` | Write tests | sonnet |
| `spec-reviewer` | Check spec compliance (READ-ONLY) | sonnet |
| `code-quality-reviewer` | Check code quality (READ-ONLY) | opus |

## Installation

Copy the `.claude/` directory into your project:

```bash
cp -r .claude/skills/ your-project/.claude/skills/
cp -r .claude/agents/ your-project/.claude/agents/
cp -r .claude/commands/ your-project/.claude/commands/
```

Then add to your project's `CLAUDE.md`:

```markdown
## Development Framework
- All skills in .claude/skills/ are MANDATORY when triggered
- Hard gates between phases (no skipping)
- Subagents get ONLY task-specific context
```

## File Structure

```
.claude/
├── commands/          # 10 slash commands
│   ├── analyze.md     # /analyze -- Phase 1
│   ├── architect.md   # /architect -- Phase 2
│   ├── design.md      # /design -- Phase 3
│   ├── decompose.md   # /decompose -- Phase 4
│   ├── validate.md    # /validate -- Phase 5
│   ├── build.md       # /build -- Phase 6
│   ├── review.md      # /review -- Phase 7
│   ├── document.md    # /document -- Phase 8
│   ├── deploy.md      # /deploy -- Phase 9
│   └── map-codebase.md # Codebase analysis utility
│
├── skills/            # 9 SKILL.md workflow definitions
│   ├── analysis/
│   ├── architecture/
│   ├── design/
│   ├── decomposition/
│   ├── validation/
│   ├── implementation/
│   ├── review/
│   ├── documentation/
│   └── deploy/
│
└── agents/            # 5 agent definitions
    ├── researcher.md
    ├── implementer.md
    ├── test-writer.md
    ├── spec-reviewer.md
    └── code-quality-reviewer.md

.planning/             # Planning artifacts (generated at runtime)
├── STATE.md           # Current feature state
├── active/            # Active feature phases (01-analysis/ through 09-deploy/)
├── framework-design.md
└── research/          # Framework research archive
```

## Inspired By

| Framework | What We Took |
|-----------|-------------|
| [Superpowers](https://github.com/obra/superpowers) | Anti-rationalization tables, `<HARD-GATE>` tags, "do not trust the report", one question per message |
| [GSD](https://github.com/gsd-build/get-shit-done) | `.planning/` as shared state, wave-based execution, deviation rules, gray areas identification |
| [Everything Claude Code](https://github.com/affaan-m/everything-claude-code) | Tool whitelist per agent, model routing, structured verdicts, handoff documents |
| [Antigravity](https://github.com/sickn33/antigravity-awesome-skills) | Understanding Lock, Decision Log, exit criteria, "3 Questions" gate |

## License

MIT
