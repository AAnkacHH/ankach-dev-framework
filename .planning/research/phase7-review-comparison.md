# Phase 7: Review — Comparison of 4 Frameworks

**Date:** 2026-03-29

---

## Key Patterns

### 1. Two-Stage Review (Superpowers)
Spec compliance FIRST → then code quality. Don't review quality of code that doesn't meet requirements.

### 2. "Do Not Trust the Report" (Superpowers)
Reviewer reads actual code, not implementer's claims. "The implementer finished suspiciously quickly."

### 3. Goal-Backward Verification (GSD) — 4 Levels
| Level | Check |
|-------|-------|
| Exists | File is present |
| Substantive | Not a stub (real logic, not placeholders) |
| Wired | Connected, imported, used |
| Data Flowing | Processes real data, not hardcoded |

### 4. Iron Law (Superpowers/Antigravity)
"No completion claims without fresh verification evidence."

### 5. Verdict Format (ECC)
APPROVE / WARNING / BLOCK with severity: CRITICAL / HIGH / MEDIUM / LOW

### 6. Pushback Encouraged (Superpowers)
Implementer should push back on review feedback when:
- Suggestion breaks existing functionality
- Reviewer lacks full context
- Violates YAGNI
- Technically incorrect for this stack

### 7. Integration Check (GSD — unique)
Cross-component: export/import maps, API route coverage, auth protection, E2E flow tracing.

### 8. Cross-AI Peer Review (GSD)
Multiple AI models (Gemini, Claude, Codex) review independently → consensus analysis.

### 9. Max Iterations
- Antigravity: 2 rounds → escalate
- GSD: 3 rounds planner↔checker
- Superpowers: no hard limit
