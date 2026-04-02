# Phase 5: Validation — Comparison of 4 Frameworks

**Date:** 2026-03-29

---

## Cross-Framework Comparison

| | **Superpowers** | **GSD** | **ECC** | **Antigravity** |
|---|---|---|---|---|
| **What validated** | Spec completeness, plan alignment, placeholders, type consistency | 10 dimensions (coverage, tasks, deps, wiring, scope, verification, context, Nyquist, cross-plan, CLAUDE.md) | Build/type/lint/test/security/diff | Issue DoD, execution gate, evidence |
| **Format** | Approved / Issues Found | VERIFICATION PASSED / ISSUES FOUND + YAML | Pass/fail per phase | Status + AC checklist + Evidence |
| **On failure** | Loop until user approves | Planner-Checker loop, max 3 iterations → human | Build fail = immediate STOP | Max 2 rounds → escalate |
| **Max iterations** | No cap | **3** | No cap (continuous) | **2** |
| **Automated vs manual** | Subagent dispatch + human approval | Automated 10-dim check + human escalation | Automated script + agent reviews | Automated gate + human for production |

## Best Patterns for Ankach Phase 5

| Pattern | Source | Why |
|---------|--------|-----|
| Requirements coverage check (every requirement → story) | GSD | Nothing falls through cracks |
| Story completeness check (AC, deps, wave, files) | GSD | Stories are ready for agents |
| Design alignment (decomposition matches Phase 3) | Superpowers (spec-plan alignment) | No drift between design and tasks |
| Scope sanity (no creep beyond Phase 1-3) | GSD | Prevents over-building |
| No placeholders / vague tasks | Superpowers | Agents can't execute "TBD" |
| Max 2 iterations then human | Antigravity | Prevents infinite loops |
| PASS/FAIL with specific issues list | GSD YAML format | Actionable feedback |
