# Phase 8-9: Documentation & Deploy — Comparison of 4 Frameworks

**Date:** 2026-03-29

---

## Phase 8: Documentation

| | Superpowers | GSD | ECC | Antigravity |
|---|---|---|---|---|
| **Doc agent?** | No | No (embedded in workflows) | Yes (doc-updater) | No (skills catalog) |
| **Auto changelog?** | No | No | No | Skill available |
| **API docs?** | N/A | N/A | Codemaps (architectural) | OpenAPI 3.1 + Postman |
| **Per-phase docs?** | None | SUMMARY.md (YAML frontmatter) | Codemaps on triggers | On demand |
| **Key pattern** | Gap — no doc automation | Machine-readable YAML frontmatter in summaries | "Generate from source of truth, not manual composition" | 8-phase doc lifecycle |

## Phase 9: Deploy

| | Superpowers | GSD | ECC | Antigravity |
|---|---|---|---|---|
| **Pre-deploy** | Iron Law (5-step evidence gate) | 5 preflight + goal-backward verification | 20+ item checklist (5 categories) | State machine + human gates |
| **Migrations** | None | None | Expand-contract pattern | None |
| **Merge strategy** | Standard merge (4 options) | Squash recommended, strips .planning/ | GitHub Flow | Not defined |
| **Post-deploy** | None | None | Canary Watch (6 dimensions) | Deploy-verify with evidence |
| **Rollback** | Branch discard | Git tags | kubectl undo, feature flags | Version-based + escalation |
