# Workflows

Two entry points depending on your use case:

| Command | Use Case | Skippable Phases |
|---------|----------|-----------------|
| `/workflow-feature <description>` | Add feature to **existing project** | Phase 2 (Architecture), Phase 8 (Docs) |
| `/workflow-full <description>` | **New project** or major feature | None |

Both walk through all 9 phases sequentially. Each phase reads the full skill definition and executes every step. You approve at every gate.

## `/workflow-feature` (Brownfield)

Optimized for adding features to existing projects:

- Automatically detects brownfield — skips greenfield detection
- Phase 2 (Architecture) can be skipped if no macro changes needed
- Phase 8 (Documentation) can be skipped if user opts out
- Codebase research focuses on existing patterns and affected areas

```
/workflow-feature Add bulk shipment endpoint

  Phase 1: Analysis .............. approve
  Phase 2: Architecture .......... skip (no macro changes)
  Phase 3: Design ................ approve
  Phase 4: Decomposition ......... approve
  Phase 5: Validation ............ PASS
  Phase 6: Implementation ........ approve (wave by wave)
  Phase 7: Review ................ SHIP
  Phase 8: Documentation ......... approve
  Phase 9: Deploy ................ done → archived
```

## `/workflow-full` (Greenfield / Major)

Full pipeline for new projects or major cross-cutting features:

- All 9 phases mandatory — none skippable
- Greenfield research via `researcher` sub-agent (stack, patterns, pitfalls)
- Full architecture decisions with ADRs
- Complete domain map from scratch

```
/workflow-full Build a webhook relay service

  Phase 1: Analysis .............. approve
  Phase 2: Architecture .......... approve (full ADRs)
  Phase 3: Design ................ approve
  Phase 4: Decomposition ......... approve
  ...all phases mandatory...
  Phase 9: Deploy ................ done → archived
```

## Individual Phase Commands

You can also run phases individually for more control:

| Command | Phase |
|---------|-------|
| `/analyze` | Phase 1 only |
| `/architect` | Phase 2 only |
| `/design` | Phase 3 only |
| `/decompose` | Phase 4 only |
| `/validate` | Phase 5 only |
| `/build` | Phase 6 only |
| `/review` | Phase 7 only |
| `/document` | Phase 8 only |
| `/deploy` | Phase 9 only |

Each individual command checks that the previous phase is approved before starting.

## User Commands During Workflow

| Command | What it does |
|---------|-------------|
| `approve` | Accept phase, move to next |
| `reject` | Stay in phase, fix issues |
| `skip` | Skip phase (only Phase 2, 8 in `/workflow-feature`) |
| `pause` | Save state, resume later |
| `abort` | Stop workflow, keep all artifacts |

## Resuming

If a workflow is interrupted:

```
/workflow-feature continue
```

Reads `.planning/STATE.md` and resumes from the current phase with full context.
