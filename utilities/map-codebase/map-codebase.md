---
description: Analyze the existing codebase and generate .context/ files + AGENTS.md/CLAUDE.md/GEMINI.md platform wrappers
---

# /map-codebase

You analyze an existing project and generate structured context files for agents and AI platforms.

**Goal:** Replace the monolithic `project-rules.md` with modular `.context/` files and create `AGENTS.md` as the universal entry point for all AI tools.

---

## Step 1: Scan the Project

Read the following files (if they exist):
- `package.json` (root + all workspace `package.json`), `composer.json`, `pyproject.toml`, `go.mod` — whatever applies
- `tsconfig.json` / `tsconfig.*.json`
- `README.md`, `CONTRIBUTING.md`
- `project-rules.md`, `CLAUDE.md`, `AGENTS.md` — existing context if any
- `.eslintrc.*`, `prettier.config.*`, `.editorconfig`, `phpcs.xml`, `.flake8`
- `jest.config.*`, `vitest.config.*`, `phpunit.xml`, `pytest.ini`

Then **deeply scan the source directory** (do not limit to 2 levels):
- Find ALL unique subdirectory patterns inside `src/` (or equivalent)
- For each domain/module directory, list ALL subdirectories it contains
- Note directories that exist at the top level (cross-cutting: Utils/, Common/, Shared/, etc.)
- Note directories that exist inside domain modules (Entity/, Service/, Repository/, DTO/, Formatter/, Enum/, Provider/, Exception/, etc.)
- Note any directories that DON'T follow the main pattern — these are important edge cases

Mark anything you are unsure about with `[NEEDS REVIEW]`.

---

## Step 2: Determine the Stack

Build an internal "stack map":
- **Language:** TypeScript / JavaScript / PHP / Python / Go / other
- **Backend:** NestJS / Symfony / Laravel / Express / Next.js / FastAPI / other (or none)
- **Frontend:** Vue / React / Angular / Svelte / other (or none)
- **Database:** PostgreSQL / MySQL / MongoDB / other
- **ORM:** TypeORM / Prisma / Doctrine / Eloquent / other
- **Auth:** JWT / OAuth / Session / other
- **Test runner:** Jest / Vitest / PHPUnit / Pytest / other
- **Lint/Format:** ESLint+Prettier / PHP-CS-Fixer / Biome / other
- **Dev/build/test commands:** extract from scripts in package files

---

## Step 3: Generate `.context/` Files

### `.context/tech-stack.md`
```markdown
# Tech Stack

## Runtime & Language
- [language and version]

## Backend
- [framework, version, key libraries]

## Frontend
- [framework, version, UI libraries]

## Database & ORM
- [DB, ORM, connection pattern]

## Authentication
- [auth mechanism]

## External Integrations
- [external APIs, services]

## Dev Commands
```bash
# Backend
[commands]

# Frontend
[commands]
```

## Misidentification Risks
Things an AI agent is likely to get wrong about this project:
- [e.g. "JS files in resources/ are utility scripts, NOT a SPA framework — frontend is server-rendered templates"]
- [e.g. "Database is X — do NOT use Y-specific syntax (specific difference examples)"]
- [e.g. "This project uses framework A v2, NOT v1 — API differences: ..."]
```

**How to populate "Misidentification Risks":**

Look for signals that commonly confuse agents:
- Project has files from multiple ecosystems (e.g. JS files in a server-rendered project → agent may assume SPA)
- Database engine has a close cousin with different SQL syntax (e.g. two databases that share syntax but differ on specific functions)
- Project uses an older or newer major version of a framework with breaking API changes
- Project has a monorepo structure where backend and frontend coexist — clarify which is which
- Project uses a less common pattern for its stack (e.g. manual mappers instead of auto-generated, custom DI instead of framework default)

If you find any of these signals, add a specific line to the Misidentification Risks section explaining what the project IS and what it is NOT. If no risks are found, write: "No significant misidentification risks detected."

### `.context/project-structure.md`

This is the most important file. Be precise — an incorrect structure misleads agents into creating wrong files.

```markdown
# Project Structure

## Repository Layout
[top-level directory tree with explanations]

## Source Structure: Cross-Cutting Layers
Directories that exist at the top of `src/` and apply across all domains:
- `src/Common/` or `src/Shared/` — [what goes here]
- `src/Utils/` — [what goes here]
- [any other top-level cross-cutting dirs]

## Source Structure: Domain Module Pattern
Each business domain lives in `src/{Layer}/{Domain}/` with these subdirectories:
| Subdirectory | Purpose | Example |
|---|---|---|
| `Entity/` | [description] | `User.php`, `order.entity.ts` |
| `Service/` | [description] | ... |
| `Repository/` | [description] | ... |
| `DTO/` | [description] | ... |
| `Formatter/` | [if exists] | ... |
| `Enum/` | [if exists] | ... |
| `Provider/` | [if exists] | ... |
| `Exception/` | [if exists] | ... |
[add ALL subdirectory types found — do not omit any]

## Exceptions to the Pattern
Domain dirs that do NOT follow the standard pattern:
- `src/{X}/` — [reason, what's different]

[ASK THE USER: list any subdirectory patterns you found but cannot confidently explain]

## File Naming Conventions
| Type | Pattern | Example |
|------|---------|---------|
[naming conventions table]

## Wiring Points (Manual Steps After Agents)
- [files that require manual registration: AppModule, routing files, etc.]
```

### `.context/coding-conventions.md`
```markdown
# Coding Conventions

## Language / Type Rules
[strict types, nullability, any usage, etc.]

## Import Order
[import ordering rules]

## Formatting
[indent, quotes, line length, trailing commas, etc.]

## Key Patterns
### Controllers / Routes
[thin controller, DTOs, validation approach]

### Services / Business Logic
[where logic lives, base classes to extend]

### [Other patterns found — Formatter, Provider, etc.]
[explain each significant pattern]

### Error Handling
[which exceptions to throw, base exception classes, hierarchy]
```

### `.context/testing-guide.md`
```markdown
# Testing Guide

## Test Structure
[where tests live, naming pattern]

## Commands
```bash
[test run commands]
```

## Unit Test Pattern
[mocking approach, describe/it or test class structure]

## Integration / E2E Pattern
[setup, teardown, database fixtures/factories]

## Test Data
[factories, fixtures, seeds — where they live and how to use]
```

### `.context/security-rules.md`
```markdown
# Security Rules

## Authentication & Authorization
[mandatory auth rules]

## Data Validation
[where and how validation happens]

## Database Security
[parameterized queries, user scoping rules]

## Secrets & Environment
[what cannot be committed, where to store secrets]

## Forbidden Actions ⛔
[list of absolute prohibitions — be specific]
```

---

## Step 4: Generate `AGENTS.md` (Universal)

This is the main file read automatically by **all** AI platforms.
Format — concise (~100 lines), only what matters most:

```markdown
# AGENTS.md — [Project Name]

## Project Overview
[1-3 sentences: what it is, for whom, main goal]

## Tech Stack
[concise list: language, BE, FE, DB, auth]

## Repository Structure
[first-level directory tree with explanations]

## Development Commands
```bash
[key commands]
```

## Architecture Rules
[5-7 most important rules an agent must follow]

## Domain Module Structure
A new domain module must contain these subdirectories:
[list ALL required subdirs based on what you found in Step 1]

## Key Conventions
[naming, imports, error handling — only the most critical]

## Testing
[commands + where tests live]

## Forbidden ⛔
[list of absolute prohibitions]

## Detailed Context
For detailed rules see `.context/` directory:
- `.context/tech-stack.md`
- `.context/project-structure.md`
- `.context/coding-conventions.md`
- `.context/testing-guide.md`
- `.context/security-rules.md`
```

---

## Step 5: Generate Thin Platform Wrappers

### `CLAUDE.md`
```markdown
# Claude Code — [Project Name]

See @AGENTS.md for all project rules and architecture.

## Stack Guards
- Database is [DB engine]. NEVER generate [other DB engine] syntax.
- Frontend is [actual frontend]. Do NOT assume [commonly confused alternative].
- [Add any other misidentification risks from .context/tech-stack.md]

## Workflow Rules
- Before implementing, present a plan and wait for approval.
- When fixing bugs, trace the actual code path first — read all involved files before proposing a fix.
- When creating diagrams, embed them inline in the document. Use Mermaid syntax only.
- NEVER run destructive commands (prune, rm -rf, force push) without explicit user confirmation.

## Claude-Specific
- Sub-agents: `.claude/agents/`
- Custom commands: `.claude/commands/`
- Planning artifacts: `.planning/`
- Detailed context: `.context/`
```

**How to populate "Stack Guards":**

Pull the concrete risks from the "Misidentification Risks" section of `.context/tech-stack.md` and turn them into direct prohibitions. CLAUDE.md is read at the start of every conversation — these guards prevent wrong assumptions before any analysis begins.

- Include only guards relevant to this specific project — do not add generic advice
- Each guard must name what TO use and what NOT to use
- If `.context/tech-stack.md` has no misidentification risks, include only the database line (most common source of errors)

### `GEMINI.md`
```markdown
# Gemini — [Project Name]

See AGENTS.md for all project rules and architecture.

## Gemini-Specific Rules
- Do NOT expand the scope of a task without explicit user approval
- Agents: `.agents/` (planner, executor, code-reviewer, implementation-verifier)
- Workflows: `.agents/workflows/`
- Planning artifacts: `.planning/`
```

---

## Step 6: Report

After generating all files, show the user:

```
✅ Generated:
  AGENTS.md              — universal (Gemini ✅ Claude ✅ Codex ✅)
  CLAUDE.md              — platform wrapper
  GEMINI.md              — platform wrapper
  .context/
    tech-stack.md        ✅
    project-structure.md ✅
    coding-conventions.md ✅
    testing-guide.md     ✅
    security-rules.md    ✅

❓ I need clarification on the following before finalizing:
  - [question 1 — specific thing you could not determine from the code]
  - [question 2 — ...]
  (If nothing is unclear, omit this section)

📁 The old project-rules.md can be archived or deleted.
```

---

## Rules

- **Scan deeply.** Do not limit directory scanning to 2 levels. Go into `src/` fully to find all subdirectory patterns.
- **Document ALL subdirectory types** found in domain modules — even uncommon ones like `Formatter/`, `Provider/`, `Enum/`. Missing these misleads agents.
- **When uncertain — ask, don't guess.** If you cannot confidently determine something from the code (e.g. the purpose of a directory, a naming convention, an architectural rule), stop and ask the user directly before writing it into the files. Do not use placeholder markers.
- **Do not duplicate** content between `AGENTS.md` and `.context/` — `AGENTS.md` contains only a summary with a reference to `.context/`
- **CLAUDE.md and GEMINI.md** — maximum 10 lines, no duplication of rules from `AGENTS.md`
- **Do NOT delete** `project-rules.md` automatically — only suggest it to the user
