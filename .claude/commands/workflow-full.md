---
description: "Full 9-phase workflow for new projects or major features. All phases, all gates."
---

Read and follow `.claude/skills/workflow-full/SKILL.md`.

For each phase, read and execute the corresponding skill in `.claude/skills/{phase}/SKILL.md` fully — all steps, all gates.

**User request:** $ARGUMENTS

**If $ARGUMENTS is "continue":**
Read `.planning/STATE.md` and resume from the current phase.

**Rules:**
- Execute ALL 9 phases sequentially
- Each phase: read its SKILL.md, follow every step
- Wait for `approve` at every HARD GATE
- Show progress tracker after each phase
- Update STATE.md after each phase completion
