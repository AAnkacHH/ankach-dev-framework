# Installation

## Option 1: Copy into your project

```bash
git clone https://github.com/AAnkacHH/ankach-dev-framework.git
cp -r ankach-dev-framework/.claude/skills/ your-project/.claude/skills/
cp -r ankach-dev-framework/.claude/agents/ your-project/.claude/agents/
cp -r ankach-dev-framework/.claude/commands/ your-project/.claude/commands/
```

## Option 2: Git submodule

```bash
cd your-project
git submodule add https://github.com/AAnkacHH/ankach-dev-framework.git .framework
# Then symlink or copy what you need
```

## Configure CLAUDE.md

Add to your project's `CLAUDE.md`:

```markdown
## Development Framework
- Skills: `.claude/skills/` — MANDATORY when triggered
- Agents: `.claude/agents/` — scoped tool access per role
- Hard gates between phases (no skipping)
- Subagents get ONLY task-specific context
- Use `/workflow-feature <description>` for new features
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
│   ├── workflow-feature.md            # /workflow-feature — brownfield
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
```

## Runtime Artifacts

The framework creates `.planning/` directory at runtime:

```
.planning/
├── STATE.md           # Current feature state (phase, status, decisions)
└── active/            # Phase artifacts
    ├── 01-analysis/
    ├── 02-architecture/
    ├── 03-design/
    ├── 04-decomposition/
    ├── 05-validation/
    ├── 06-implementation/
    ├── 07-review/
    ├── 08-documentation/
    └── 09-deploy/
```

On completion, artifacts are archived to `docs/features/YYYY-MM-DD-{slug}/`.

## Requirements

- [Claude Code](https://claude.ai/code) CLI or Desktop app
- Git (for feature branches, commits, PRs)
- Your project's test runner (for Phase 6-7 verification)
