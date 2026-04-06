---
description: "Feature documentation pipeline. Parallel generation of requirements, use cases, and diagrams."
---

Read and follow `.claude/skills/doc-pipeline/SKILL.md` step by step.

**User request:** $ARGUMENTS

**Rules:**
- Confirm tech stack before research.
- Ask questions ONE at a time, max 7.
- Understanding Lock MUST be confirmed before generating.
- Spawn 3 agents IN PARALLEL for generation step.
- Validate Mermaid syntax before finalizing.
- Wait for approval at HARD GATE.
- Write artifacts to `.planning/active/docs-{slug}/`
