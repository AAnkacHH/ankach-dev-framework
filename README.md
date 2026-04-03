<img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=6,11,20&height=180&section=header&text=Ankach%20AI%20Workflows&fontSize=36&fontColor=fff&animation=twinkling&fontAlignY=32&desc=Structured%20AI-assisted%20development%20workflows%20for%20Claude%20Code&descAlignY=52&descSize=16" width="100%"/>

<p align="center">
  <a href="https://github.com/AAnkacHH/ankach-dev-framework/stargazers"><img src="https://img.shields.io/github/stars/AAnkacHH/ankach-dev-framework?style=flat-square&color=6366F1&labelColor=1e1e2e" alt="Stars"/></a>
  <a href="https://github.com/AAnkacHH/ankach-dev-framework/network/members"><img src="https://img.shields.io/github/forks/AAnkacHH/ankach-dev-framework?style=flat-square&color=a78bfa&labelColor=1e1e2e" alt="Forks"/></a>
  <a href="https://github.com/AAnkacHH/ankach-dev-framework/blob/main/LICENSE"><img src="https://img.shields.io/github/license/AAnkacHH/ankach-dev-framework?style=flat-square&color=22c55e&labelColor=1e1e2e" alt="License"/></a>
  <a href="https://claude.ai/code"><img src="https://img.shields.io/badge/Claude_Code-compatible-6366F1?style=flat-square&labelColor=1e1e2e" alt="Claude Code"/></a>
</p>

<p align="center">
  <a href="#-quick-start"><img src="https://img.shields.io/badge/Quick_Start-4285F4?style=flat-square"/></a>
  <a href="#-workflows"><img src="https://img.shields.io/badge/Workflows-22c55e?style=flat-square"/></a>
  <a href="#-utilities"><img src="https://img.shields.io/badge/Utilities-a78bfa?style=flat-square"/></a>
  <a href="docs/patterns.md"><img src="https://img.shields.io/badge/Patterns-f59e0b?style=flat-square"/></a>
  <a href="docs/examples.md"><img src="https://img.shields.io/badge/Examples-ef4444?style=flat-square"/></a>
</p>

---

## The Problem

AI coding assistants are powerful but chaotic. They skip analysis, jump to code, miss edge cases, and produce inconsistent results.

**This framework fixes that** by adding structure, documentation, and human control at every step.

<table>
<tr>
<td width="50%">

### Without framework

- Jumps straight to code
- Skips architecture decisions
- No documentation trail
- Inconsistent quality
- Hard to resume after interruption

</td>
<td width="50%">

### With framework

- Structured phases with approval gates
- Every decision documented
- Anti-rationalization tables
- Parallel agent execution
- Survives context resets

</td>
</tr>
</table>

> [!TIP]
> Built by combining the best patterns from [Superpowers](https://github.com/obra/superpowers), [GSD](https://github.com/gsd-build/get-shit-done), [Everything Claude Code](https://github.com/affaan-m/everything-claude-code), and [Antigravity Awesome Skills](https://github.com/sickn33/antigravity-awesome-skills).

---

## Quick Start

```bash
# Clone the framework
git clone https://github.com/AAnkacHH/ankach-dev-framework.git

# Copy a workflow into your project's .claude/ directory
# (installer tool coming soon)
```

Then in Claude Code:

```
/ankach:workflow-feature Add user authentication with OAuth
```

---

## Workflows

Each workflow is a **self-contained folder** with its own skills, commands, agents, and orchestrator. Click through for details.

| Workflow | Description | Skills | Agents | Commands |
|----------|-------------|:------:|:------:|:--------:|
| [**dev-pipeline**](workflows/dev-pipeline/) | 9-phase development: `/ankach:analyze` → `/ankach:deploy` | 9 | 5 | 11 |
| [**research-analysis**](workflows/research-analysis/) | 9-step paper analysis: `/ankach:process` → `/ankach:so-what` | 9 | — | 10 |
| **client-discovery** | Analyze a client's business, model, competitors, and SEO before proposing a solution | — | — | — |

> [!NOTE]
> Adding a new workflow = adding a new folder to `workflows/`. Each workflow is independent and has its own README with full documentation.

### Coming Soon

<details>
<summary><strong>client-discovery</strong> — understand the client before writing a single line of code</summary>

When a client says "I need a website" or "I need a service" but doesn't explain **why** — this workflow fills in the gaps before you propose anything.

**Planned phases:**
- **Company Profile** — what the company does, what it sells, who its customers are
- **Business Model Analysis** — how the company makes money, value proposition, revenue streams
- **Market & Competitor Analysis** — who the competitors are, what they do better/worse, market positioning
- **SEO & Online Presence Audit** — current search visibility, keyword opportunities, content gaps
- **Requirements Synthesis** — translate business understanding into a concrete technical proposal

**Goal:** Never propose a generic "5-page website" again. Understand the business first, then recommend what they actually need.

</details>

---

## Utilities

Standalone tools that are not part of any workflow.

| Utility | Description |
|---------|-------------|
| [**init-project**](utilities/init-project/) | Initialize a new project — Q&A, then generate `.context/`, `AGENTS.md`, `CLAUDE.md`, `GEMINI.md` |
| [**map-codebase**](utilities/map-codebase/) | Analyze an existing codebase and generate `.context/` files |

---

## Repository Structure

```
workflows/                         # Each workflow is a self-contained unit
├── dev-pipeline/                  # → README.md inside with full docs
│   ├── skills/                    # Phase skills (SKILL.md each)
│   ├── commands/                  # Slash commands
│   └── agents/                    # Sub-agents
├── {your-workflow}/               # Add new workflows here
│
utilities/                         # Standalone tools
├── map-codebase/
│
docs/                              # Framework-level documentation
```

---

## Conventions

- **Diagrams:** All diagrams use [Mermaid](https://mermaid.js.org/) syntax — never ASCII art. Before generating, verify current syntax via [context7](https://context7.com) (`/mermaid-js/mermaid`). See [Patterns](docs/patterns.md#mermaid-diagrams-all-phases) for details.

---

## Documentation

| Document | Description |
|----------|-------------|
| [Patterns](docs/patterns.md) | Hard gates, anti-rationalization, validation loops, wave execution |
| [Examples](docs/examples.md) | Usage examples: brownfield, greenfield, single phase, resume |
| [Installation](docs/installation.md) | Setup guide and file structure |
| [Inspiration](docs/inspiration.md) | What we took from Superpowers, GSD, ECC, Antigravity |

---

## Contributing

Contributions are welcome! To add a new workflow:

1. Fork the repository
2. Create a folder in `workflows/` with `README.md`, `skills/`, `commands/`, `agents/`
3. Follow the existing structure
4. Open a PR

---

<p align="center">
  <sub>MIT &copy; 2026 <a href="https://github.com/AAnkacHH">Andrii Plyskach</a></sub>
</p>

<img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=6,11,20&height=100&section=footer" width="100%"/>
