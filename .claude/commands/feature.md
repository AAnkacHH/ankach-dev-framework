---
description: "Add a feature to existing project. Walks through all 9 phases sequentially with approval gates."
---

Read and follow `.claude/skills/feature/SKILL.md` step by step.

**Feature request:** $ARGUMENTS

**If $ARGUMENTS is "continue":**
Read `.planning/STATE.md` and resume from the current phase.

**Rules:**
- Follow ALL 9 phases in order. Do not skip unless user says `skip`.
- Wait for `approve` at every phase gate.
- Each phase reads its corresponding SKILL.md for detailed instructions.
- Update STATE.md after each phase completion.
- Show progress tracker after each phase.
