---
name: cleanup-branches
description: Find and delete local branches that are safely merged into the prod branch (main, or master on legacy repos), plus prune stale remote-tracking refs. Always shows the list first and requires explicit confirmation before deleting. Invoke when the user wants to clean up local git state, delete merged branches, or get rid of old feature branches.
allowed-tools:
  - Bash(git status)
  - Bash(git status --porcelain*)
  - Bash(git branch)
  - Bash(git branch -a*)
  - Bash(git branch -r*)
  - Bash(git branch -v*)
  - Bash(git branch --show-current)
  - Bash(git branch --list*)
  - Bash(git branch --merged*)
  - Bash(git branch --no-merged*)
  - Bash(git branch -d *)
  - Bash(git fetch --prune*)
  - Bash(git fetch origin*)
  - Bash(git remote prune*)
  - Bash(git for-each-ref*)
  - Bash(git symbolic-ref*)
  - Bash(git rev-parse*)
  - Bash(git rev-list*)
  - Bash(git log --oneline*)
  - Bash(git log -1*)
  - Bash(git log * --not *)
  - Bash(git ls-files*)
  - Bash(git show-ref*)
  - AskUserQuestion
disallowed-tools:
  # force / broad delete (git branch -d * is in allow — hard-deny destructive variants)
  - Bash(git branch -D*)
  - Bash(git branch --delete --force*)
  - Bash(git branch --delete --remotes*)
  - Bash(git branch -dr*)
  - Bash(git branch -Dr*)
  - Bash(git branch -rd*)
  - Bash(git branch -rD*)
  # default-branch hard-blocks (belt-and-suspenders beyond prose)
  - Bash(git branch -d main)
  - Bash(git branch -d main *)
  - Bash(git branch -d master)
  - Bash(git branch -d master *)
  - Bash(git branch -d dev)
  - Bash(git branch -d dev *)
  - Bash(git branch -d stage)
  - Bash(git branch -d stage *)
  - Bash(git branch -d release/*)
  - Bash(git branch -d hotfix/*)
  # ref manipulation / destructive ops
  - Bash(git update-ref*)
  - Bash(git filter-branch*)
  - Bash(git filter-repo*)
  - Bash(git replace*)
  - Bash(git fsck*)
  - Bash(git gc*)
  - Bash(git reflog expire*)
  - Bash(git reset*)
  - Bash(git clean*)
  - Bash(git checkout*)
  - Bash(git switch*)
  - Bash(git commit*)
  - Bash(git merge*)
  - Bash(git rebase*)
  - Bash(git config*)
  - Bash(git remote add*)
  - Bash(git remote remove*)
  - Bash(git remote rm*)
  - Bash(git remote set-url*)
---

# Cleanup Merged Branches

Remove local branches that are safely merged into the default branch, and optionally prune stale remote-tracking references. Never force-delete unmerged branches; never touch remote branches.

## Procedure

### 1. Preflight

```bash
git status
git branch --show-current
git fetch --prune origin
```

- If the tree is dirty → warn, but proceed (cleanup doesn't modify the working tree). Ask before continuing if there are unpushed commits on the current branch.
- Detect default branch — try `git symbolic-ref` first; if it fails (no HEAD ref on a fresh clone or the symbolic-ref was never set locally), fall back to `for-each-ref` preferring `main` over `master`:

  ```bash
  # primary
  git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's|^refs/remotes/origin/||'

  # fallback (if primary printed nothing)
  git for-each-ref --format='%(refname:short)' 'refs/remotes/origin/*' \
    | grep -E '^origin/(main|master)$' | head -1 | sed 's|^origin/||'
  ```

  If both commands fail, stop and ask the user — do not guess.

### 2. Find merged branches

List local branches fully merged into the default branch, excluding:

- The default branch itself.
- The current branch (cannot be deleted while checked out).
- `release/*`, `hotfix/*` currently in flight (if any) — list them but do not suggest deletion.

```bash
git branch --merged origin/<default> \
  | sed 's/^[* +]*//' \
  | grep -vxE '^(main|master|dev|stage|release/.*|hotfix/.*)$' \
  | grep -vxF "$current_branch"
```

- `sed 's/^[* +]*//'` strips the leading current-branch marker (`*`), worktree marker (`+ `), and leading spaces — without this, anchored patterns would miss branches that are checked out in another worktree.
- `grep -vxE` uses extended regex with full-line match (`-x`) to exclude the protected set precisely.
- `grep -vxF "$current_branch"` uses fixed-string (`-F`) full-line match (`-x`) — critical because branch names may contain `.`, `+`, or other regex metacharacters that would turn a plain `grep -v` into accidental wildcard matching (e.g. a branch literally named `main.backup` would otherwise match `main.`).

The protected set is `main|master|dev|stage|release/*|hotfix/*`.

Also collect metadata for the report:

- Last commit date per branch: `git log -1 --format='%ci %s' <branch>`
- Whether the branch was pushed and its remote is gone (i.e. PR merged + remote deleted): `git for-each-ref --format='%(refname:short) %(upstream:track)' refs/heads/ | grep '\[gone\]'`

### 3. Classify candidates

Group into three buckets:

1. **Safe to delete** — local-only or tracking `[gone]` remote, fully merged.
2. **Keep** — current branch, default branch, protected patterns (`release/*`, `hotfix/*` unless user overrides).
3. **Ambiguous** — merged into `origin/<default>` but remote still exists. List separately; some teams keep the remote for traceability.

### 4. Show the plan (dry run)

Present a table in your output:

```
Branches to delete (merged into origin/main):
  feature/PROJ-123-availability   2025-04-02  [gone]
  bugfix/ABC-42-price-calc        2025-03-28  [gone]

Ambiguous (merged, but remote still exists):
  feature/PROJ-125-translations   2025-04-10  tracks origin/feature/PROJ-125-translations

Protected / skipped:
  main (default)
  feature/PROJ-900-active (current)
  hotfix/PROJ-950-payment (protected pattern)
```

Then ask via `AskUserQuestion`:

- **Delete safe list only** — proceed with bucket 1.
- **Delete safe + ambiguous** — include bucket 3 (user confirms).
- **Pick individually** — ask again, listing each branch.
- **Abort** — stop.

### 5. Delete

For each confirmed branch:

```bash
git branch -d <branch>          # uses lowercase -d (safe, requires merge)
```

- **Never use `-D`** (force-delete) in this skill. If `-d` fails because git reports "not fully merged", stop and ask — the branch may have unreachable commits.
- Collect results: deleted OK, skipped, errored.

### 6. Prune remote-tracking refs (optional)

The `git fetch --prune` in step 1 already removes stale `origin/*` refs. If the user asks for a deeper clean, show:

```bash
git remote prune origin --dry-run
```

And ask before applying without `--dry-run`.

### 7. Report

**Success:** counts + list of deleted.

```text
deleted 5: feature/PROJ-123, bugfix/ABC-42, feature/PROJ-125, bugfix/ABC-60, feature/PROJ-130
skipped 2 (live remote)
pruned 3 remote-tracking refs
```

Omit lines that are zero.

**Failure:** full context.

- Which branch failed `git branch -d` (stderr).
- Why (not fully merged? upstream still present?).
- List of already-deleted branches in this run (so caller knows the partial state).
- Recovery: "inspect `git log <branch> --not origin/main`; then either commit missing changes or force-delete manually".

## Rules

- **Never `-D` force-delete** unmerged branches. If a branch isn't merged, stop and ask.
- **Never delete the current branch** — checkout default first only if user explicitly confirms.
- **Never delete the default branch** (`main`, or `master` on legacy repos).
- **Never touch `release/*` or `hotfix/*`** without explicit opt-in.
- **Never delete remote branches** — strictly out of scope. This includes all variants:
  - `git push origin --delete <branch>` / `git push origin :<branch>`
  - `git branch -dr origin/<branch>` / `git branch -rD origin/<branch>`
  - `git update-ref -d refs/remotes/origin/<branch>`
  - Any other command that removes refs from `origin` or any other remote.
  The `disallowed-tools` list blocks these patterns. If a user asks to delete a remote branch, refuse and tell them to do it manually via their git host's UI (GitHub → Branches, Bitbucket → Branches) or a one-off command they run themselves. This skill only deletes **local** branches and prunes **stale remote-tracking refs** (pointers to branches already deleted on the remote — those are harmless to remove via `git fetch --prune`).
- **`git fetch --prune` is allowed** — it only removes local pointers to remote branches that no longer exist upstream. It never touches the remote itself.
- **Always dry-run first**: show the list, ask, then delete.

## Examples

### Typical cleanup

1. `git fetch --prune origin`.
2. Found 7 merged local branches; 5 track `[gone]` remotes, 2 have live remotes.
3. Show table, ask.
4. User: "delete safe list only".
5. `git branch -d` for each of the 5 → all succeed.
6. Report: 5 deleted, 2 skipped (live remote), current branch preserved.

### Unmerged found

1. One candidate appears merged by `--merged` but `git branch -d` fails ("not fully merged").
2. Stop, report the branch, suggest: inspect with `git log <branch> --not origin/main` before force-deleting manually.
