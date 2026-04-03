# Dev Pipeline

> 9-phase development workflow: from idea to deploy with manual approval gates at every step.

```mermaid
graph LR
    A["/analyze"] --> B["/architect"] --> C["/design"] --> D["/decompose"]
    D --> E["/validate"] --> F["/build"] --> G["/review"] --> H["/document"] --> I["/deploy"]
```

## Orchestrators

| File | Use When |
|------|----------|
| [workflow-full.md](workflow-full.md) | New project or major feature from scratch (greenfield) |
| [workflow-feature.md](workflow-feature.md) | Adding a feature to an existing project (brownfield) |

## The 9-Phase Pipeline

```mermaid
graph TD
    P1["1. Analysis<br/><i>Understand the task, research, propose approaches</i>"]
    P2["2. Architecture<br/><i>Macro: servers, communication, technologies</i>"]
    P3["3. Design<br/><i>Micro: domain map, entities, API contracts</i>"]
    P4["4. Decomposition<br/><i>EPIC → STORIES → TASKS with wave plan</i>"]
    P5["5. Validation<br/><i>Architecture review + requirements coverage</i>"]
    P6["6. Implementation<br/><i>Parallel agents, wave-by-wave execution</i>"]
    P7["7. Review<br/><i>Spec compliance → Code quality → Tests</i>"]
    P8["8. Documentation<br/><i>API docs, dev notes, changelog</i>"]
    P9["9. Deploy<br/><i>Checklist, merge/PR, verify, archive</i>"]

    P1 --> P2 --> P3 --> P4 --> P5 --> P6 --> P7 --> P8 --> P9
    P5 -- "FAIL" --> P3
    P5 -- "FAIL" --> P4
    P7 -- "NEEDS WORK" --> P4
    P7 -- "NEEDS WORK" --> P5
    P7 -- "NEEDS WORK" --> P6
```

## Skills

| # | Phase | Skill | Command |
|---|-------|-------|---------|
| 1 | Analysis | [analysis/SKILL.md](skills/analysis/SKILL.md) | [/analyze](commands/analyze.md) |
| 2 | Architecture | [architecture/SKILL.md](skills/architecture/SKILL.md) | [/architect](commands/architect.md) |
| 3 | Design | [design/SKILL.md](skills/design/SKILL.md) | [/design](commands/design.md) |
| 4 | Decomposition | [decomposition/SKILL.md](skills/decomposition/SKILL.md) | [/decompose](commands/decompose.md) |
| 5 | Validation | [validation/SKILL.md](skills/validation/SKILL.md) | [/validate](commands/validate.md) |
| 6 | Implementation | [implementation/SKILL.md](skills/implementation/SKILL.md) | [/build](commands/build.md) |
| 7 | Review | [review/SKILL.md](skills/review/SKILL.md) | [/review](commands/review.md) |
| 8 | Documentation | [documentation/SKILL.md](skills/documentation/SKILL.md) | [/document](commands/document.md) |
| 9 | Deploy | [deploy/SKILL.md](skills/deploy/SKILL.md) | [/deploy](commands/deploy.md) |

## Agents

5 specialized sub-agents, each with a fresh context window:

| Agent | Role |
|-------|------|
| [**Implementer**](agents/implementer.md) | Implements a single story with sub-tasks |
| [**Researcher**](agents/researcher.md) | Deep research for analysis and architecture phases |
| [**Spec Reviewer**](agents/spec-reviewer.md) | Checks implementation matches design spec |
| [**Code Quality Reviewer**](agents/code-quality-reviewer.md) | Checks SOLID, security, performance, conventions |
| [**Test Writer**](agents/test-writer.md) | Writes unit and integration tests |

## Key Patterns

<details>
<summary><strong>Hard Gates</strong> — every phase requires explicit human approval</summary>

```xml
<HARD-GATE>
Wait for explicit approval before proceeding.
On approve → update STATE.md, move to next phase.
On reject → fix and re-present.
</HARD-GATE>
```

No phase can be skipped. No rationalization accepted.

</details>

<details>
<summary><strong>Anti-Rationalization Tables</strong> — prevent agents from cutting corners</summary>

Every skill includes a table of excuses the agent might generate and why each is invalid:

| Excuse | Reality |
|--------|---------|
| "This is too simple for analysis" | Simple features that touch external systems need analysis. |
| "Let's just start coding" | Architecture decisions made during coding are never revisited. |
| "We already use this pattern" | Confirm it applies here. Existing ≠ always correct. |

</details>

<details>
<summary><strong>Validation Loops</strong> — Phase 5 and 7 can loop back</summary>

```mermaid
graph LR
    P5["Phase 5: Validation"] -- "FAIL" --> P3["Phase 3: Design"]
    P5 -- "FAIL" --> P4["Phase 4: Decomposition"]
    P7["Phase 7: Review"] -- "NEEDS WORK" --> P6["Phase 6: Implementation"]
    P7 -- "NEEDS WORK" --> P5
    P7 -- "NEEDS WORK" --> P4
```

Max 2 iterations per loop. After 2 failures, escalate to the user.

</details>

<details>
<summary><strong>Wave-Based Parallel Execution</strong> — stories run in parallel agents</summary>

```mermaid
graph TD
    subgraph WaveA["Wave A — foundation, no deps"]
        S1["Story 1"]
    end
    subgraph WaveB["Wave B — parallel"]
        S2["Story 2"]
        S3["Story 3"]
    end
    subgraph WaveC["Wave C — depends on Wave B"]
        S4["Story 4"]
    end
    S1 --> S2
    S1 --> S3
    S2 --> S4
    S3 --> S4
```

Each agent gets a fresh context window — no context pollution between stories.

</details>
