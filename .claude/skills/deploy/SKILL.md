---
name: deploy
description: >
  Phase 9: Deploy — pre-deploy checklist, migrations, merge/PR, post-deploy verification,
  archive planning artifacts. Final phase of the pipeline.
---

# Phase 9: Deploy

<HARD-GATE>
Requires approved Phase 8 Documentation.
Read `.planning/active/08-documentation/index.md` — if not APPROVED, stop.
</HARD-GATE>

---

## Step 1: Context (Silent)

Read all phase artifacts to prepare deployment summary.

---

## Step 2: Pre-Deploy Checklist

Verify each item:

- [ ] All tests pass (`dotnet test`)
- [ ] Phase 7 review verdict = SHIP
- [ ] No CRITICAL issues in any review
- [ ] No hardcoded secrets in code
- [ ] Database migrations tested (if any)
- [ ] Documentation updated (Phase 8)
- [ ] Changelog entry written
- [ ] Environment variables identified and documented

Write to `.planning/active/09-deploy/pre-deploy-checklist.md`.

**If any item FAILS → STOP and report.**

---

## Step 3: Migration Plan (if DB changes)

If Phase 3 data-model.md includes schema changes:

```markdown
# Migration Plan

## Migrations
1. {migration}: {what it does}

## Strategy
{Additive only / Expand-contract for breaking changes}

## Rollback
{How to rollback if migration fails}

## Testing
- [ ] Tested on dev database
- [ ] No data loss on existing records
```

Write to `.planning/active/09-deploy/migration-plan.md`.
Skip if no DB changes.

---

## Step 4: Merge / PR

Present options to user:

1. **Create PR** (recommended) — push branch, create PR with summary
2. **Merge locally** — merge to main locally
3. **Keep branch** — don't merge yet

**If PR:** create with summary from all phases:

```bash
gh pr create --title "{Feature Name}" --body "$(cat <<'EOF'
## Summary
{from Phase 1 analysis}

## Changes
{from Phase 6 implementation summary}

## Testing
{from Phase 7 test results}

## Documentation
{from Phase 8}

## Checklist
- [x] All tests pass
- [x] Code review: SHIP
- [x] Documentation updated
- [x] Changelog entry written
EOF
)"
```

---

## Step 5: Post-Deploy Verification

After merge/deploy:

- [ ] Application starts without errors
- [ ] Health check endpoint responds
- [ ] Key user flows work (smoke test)
- [ ] No new errors in logs

Write to `.planning/active/09-deploy/post-deploy-verification.md`.

---

## Step 6: Archive + Approval

### 1. Archive planning artifacts

Copy `.planning/active/` → `docs/features/{YYYY-MM-DD}-{slug}/`

### 2. Clean up

Clear `.planning/active/` for next feature.
Update `.planning/STATE.md` → feature completed.

### 3. Present final summary

```
Feature "{name}" deployed successfully.
- {N} stories, {M} commits
- Review: SHIP
- Archived to docs/features/{date}-{slug}/

Pipeline complete.
```

<HARD-GATE>
Wait for user confirmation before archiving.
</HARD-GATE>

---

## Exit Criteria (ALL must be true)

- [ ] Pre-deploy checklist all items PASS (Step 2)
- [ ] Migration plan written if DB changes (Step 3)
- [ ] Merge/PR completed (Step 4)
- [ ] Post-deploy verification passed (Step 5)
- [ ] Planning artifacts archived to docs/features/ (Step 6)
- [ ] STATE.md updated — feature completed

---

## Anti-Rationalization Table

| Excuse | Reality |
|--------|---------|
| "Tests passed, deploy is safe" | Tests in dev ≠ behavior in production. Follow the checklist. |
| "Migration is straightforward, no plan needed" | Straightforward migrations still need rollback plans. |
| "Let's skip post-deploy verification, it's late" | Late-night deploys without verification = midnight incidents. |
