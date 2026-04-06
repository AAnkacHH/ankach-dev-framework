# Future Improvements

Based on Claude Code usage analysis (1,575 messages / 90 sessions / 2026-03-06 to 2026-04-06).

Points 3, 4, 5, 6, 9 are done. Remaining:

---

## 1. Lightweight Workflows (bug-fix, daily-ops)

### Bug-Fix Workflow

**Problem:** 15 bug-fix sessions don't need 9 phases. Claude guesses at code paths → wrong fix → multiple debugging rounds.

**Proposal:** 3-phase lightweight workflow in `workflows/bug-fix/`:

| Step | Name | What Happens |
|------|------|-------------|
| 1 | Trace | Spawn read-only sub-agent that maps the full call chain. HARD GATE before proceeding. |
| 2 | Fix | Implement fix using the trace as context. Run linter hooks. |
| 3 | Verify | Run test suite, check regressions, create commit. |

**Key agent:** `tracer.md` — read-only agent (Read, Grep, Glob only) that traces a complete code path and returns a call-chain document.

**Addresses:** 48 wrong-approach incidents, 15 bug-fix sessions with recurring "wrong code path" friction.

### Daily-Ops Workflow

**Problem:** 7 Jira planning sessions + 7 Obsidian sessions are repetitive but not covered by any workflow.

**Proposal:** Lightweight workflow in `workflows/daily-ops/` with commands:
- `/ankach:daily-plan` — Create Jira daily plan with subtasks, review yesterday's progress
- `/ankach:log` — Log activity to Obsidian vault (walk, sport, reading, etc.)
- `/ankach:standup` — Generate standup summary from Jira + git activity

**Note:** Depends on MCP integrations (Jira, Obsidian). Design should be generic enough to work with different task trackers and note systems.

---

## 2. Hooks Infrastructure

**Problem:** 39 buggy-code incidents (wrong SQL dialect, broken syntax, wrong mapper patterns). Listed as "pending" in framework design.

**Proposal:** Add `utilities/hooks/` with ready-to-copy hook configurations:

```
utilities/hooks/
├── README.md              # How to install and configure hooks
├── php-lint.json          # php -l after Edit/Write on *.php
├── typescript-check.json  # tsc --noEmit after Edit/Write on *.ts
├── mermaid-validate.json  # mmdc validate after diagram edits
└── dotnet-build.json      # dotnet build after *.cs edits
```

Each file should be a `.claude/settings.json` snippet that users can copy into their project. Include:
- `postToolUse` hooks that trigger on `Edit|Write` for specific file extensions
- Instructions on how to merge with existing settings
- Examples of custom hooks for other languages/frameworks

**Addresses:** 39 buggy-code incidents. PHP lint hook alone would catch a significant chunk.

---

## 7. Bug-Fix Workflow (Detailed Design)

Merged with point 1 above. See "Bug-Fix Workflow" section.

---

## 8. Client-Discovery Workflow

**Problem:** Listed as "coming soon" in README but not built. Natural fit for B2B platform work (8 sessions).

**Proposal:** Implement the 5 phases already planned in README:

| Phase | Name | Purpose |
|-------|------|---------|
| 1 | Company Profile | What the company does, sells, who customers are |
| 2 | Business Model Analysis | How it makes money, value proposition, revenue streams |
| 3 | Market & Competitor Analysis | Competitors, positioning, what they do better/worse |
| 4 | SEO & Online Presence Audit | Search visibility, keyword opportunities, content gaps |
| 5 | Requirements Synthesis | Translate business understanding into concrete technical proposal |

**Goal:** Never propose a generic "5-page website" again. Understand the business first.

**Agents to consider:**
- `market-researcher` — web search for competitors, market data
- `seo-auditor` — analyze online presence, keyword gaps
- `proposal-writer` — synthesize findings into a technical proposal

---

## Priority Order

| # | Improvement | Effort | Impact |
|---|---|---|---|
| 2 | Hooks infrastructure | Low | High |
| 1a | Bug-fix workflow | Medium | High |
| 1b | Daily-ops workflow | Medium | Medium |
| 8 | Client-discovery workflow | High | Medium |
