---
description: "Add feature to existing project. 9 phases with brownfield optimizations and skippable gates."
---

Read and follow `.claude/skills/workflow-feature/SKILL.md`.

For each phase, read and execute the corresponding skill in `.claude/skills/{phase}/SKILL.md` fully — all steps, all gates.

**Feature request:** $ARGUMENTS

**If $ARGUMENTS is "continue":**
Read `.planning/STATE.md` and resume from the current phase.

**Rules:**
- Execute ALL 9 phases sequentially (Phase 2 and 8 skippable)
- Each phase: read its SKILL.md, follow every step
- Wait for `approve` at every HARD GATE
- Show progress tracker after each phase
- Apply brownfield optimizations from workflow-feature/SKILL.md
- Update STATE.md after each phase completion
