---
name: sync-branch
description: Sync the current working branch with its correct long-lived base (main/master or dev/stage), by rebase or merge. Picks base by branch type (hotfix → main/master; feature/bugfix → ask dev/stage vs main/master). Always asks which strategy to use. Stops cleanly on conflicts without forcing anything. Invoke when the user wants to update a branch, pull in upstream changes, or resolve "branch is behind" situations.
allowed-tools:
  - Bash(git status)
  - Bash(git status --porcelain*)
  - Bash(git branch --show-current)
  - Bash(git branch -r*)
  - Bash(git branch -a*)
  - Bash(git fetch origin)
  - Bash(git fetch origin *)
  - Bash(git fetch --all*)
  - Bash(git fetch --prune*)
  - Bash(git rev-list*)
  - Bash(git rev-parse*)
  - Bash(git symbolic-ref*)
  - Bash(git for-each-ref*)
  - Bash(git rebase origin/*)
  - Bash(git rebase --continue)
  - Bash(git rebase --abort)
  - Bash(git rebase --skip)
  - Bash(git merge origin/*)
  - Bash(git merge --no-ff origin/*)
  - Bash(git merge --ff-only origin/*)
  - Bash(git merge --abort)
  - Bash(git stash push*)
  - Bash(git stash pop*)
  - Bash(git stash list*)
  - Bash(git stash show*)
  - Bash(git diff --name-only*)
  - Bash(git log --oneline*)
  - Bash(git log -1*)
  - AskUserQuestion
disallowed-tools:
  # dangerous git (defense-in-depth)
  - Bash(git update-ref*)
  - Bash(git filter-branch*)
  - Bash(git filter-repo*)
  - Bash(git replace*)
  - Bash(git fsck*)
  - Bash(git gc*)
  - Bash(git reflog expire*)
  - Bash(git config*)
  - Bash(git remote add*)
  - Bash(git remote remove*)
  - Bash(git remote rm*)
  - Bash(git remote set-url*)
  # narrows broader allow patterns
  - Bash(git rebase * --exec*)
  - Bash(git stash drop*)
  - Bash(git stash clear*)
  - Bash(git branch -d*)
  - Bash(git branch -D*)
  - Bash(git branch --delete*)
  - Bash(git commit*)
---

# Sync Branch with Its Base

Bring the current working branch up to date with the correct long-lived base branch. **Two decisions must be explicit: which base, and which strategy (rebase vs merge).** Never default silently.

## Base-branch policy

Production branch is **`main`** (modern default). **`master`** still exists on older repos for historical reasons — treat them as equivalent; detect which one lives on `origin`.

Base depends on the **type** of the current branch (prefix before `/`):

- **`hotfix/*`** → `main` only (or `master` on legacy repos). Strict — urgent prod fix syncs against prod.
- **`feature/*`** / **`bugfix/*`** → could be either:
  - `dev` / `stage` — if the project uses a staging flow (feature merges to stage first, then promoted to prod).
  - `main` (or `master` on legacy repos) — if the project skips stage, or if the work must be rebased onto prod directly.
  - **Ask the user** — the project's flow decides, not this skill.
- **Other prefixes** (e.g. `release/*`, custom) → ask the user explicitly.

Detect which long-lived branches exist before asking. Use `for-each-ref` — `git branch -r | grep` includes the `origin/HEAD -> origin/main` arrow line and leading whitespace that break the anchor:

```bash
git for-each-ref --format='%(refname:short)' 'refs/remotes/origin/*' \
  | grep -E '^origin/(main|master|dev|stage)$'
```

Never hard-code a production branch name without verifying it exists. If both `main` and `master` exist on the same repo (rare, mid-migration), prefer `main` and ask the user about the intended retirement plan.

## Procedure

### 1. Inspect state

```bash
git status              # never use -uall
git branch --show-current
git fetch origin
git for-each-ref --format='%(refname:short)' 'refs/remotes/origin/*' \
  | grep -E '^origin/(main|master|dev|stage)$'
```

Capture:

- **Current branch** — must NOT be a long-lived branch. If it is, stop and explain.
- **Branch type** — extract prefix (`feature` / `bugfix` / `hotfix` / other).
- **Available long-lived branches** on `origin`.
- **Dirty tree?** — any staged, unstaged, or untracked tracked changes.

### 2. Early exits

- Current branch is `master` / `main` / `dev` / `stage` / `release/*` → stop. Sync-against-itself is not this skill's job; suggest `git pull --ff-only`.

### 3. Pick the base branch

Apply the base-branch policy:

- **Branch type = `hotfix`** → base is `main` / `master` (pick whichever exists on `origin`). Do not ask — this is strict.
- **Branch type = `feature` / `bugfix`** → ask the user via `AskUserQuestion`:
  - Options come from available long-lived branches on `origin`. Typical pairing:
    - "**`dev` / `stage`** (recommended for feature/bugfix — staging flow)"
    - "**`main` / `master`** (skip staging; rebase directly onto prod)"
    - "Abort"
  - If only one long-lived branch exists → use it, but **still confirm** with a short question so the user knows.
- **Branch type = other** → ask the user explicitly which base to use.
- **User overrides** (e.g. "sync with main") → honor it after one sanity check: if it contradicts the type (e.g. `hotfix` vs `dev`), warn and require confirmation.

Store the result as `<base>` — used for all remaining steps.

### 4. Compute divergence

**Preflight — verify `origin/<base>` exists locally:**

```bash
git rev-parse --verify origin/<base> >/dev/null 2>&1 || {
  # origin/<base> not known locally — fail fast
  echo "origin/<base> does not exist (fetch it or pick a different base)"
  # stop, report to user
}
```

If `origin/<base>` is not known locally (e.g. the user chose a base that exists on origin but has never been fetched to this clone), stop and ask the user to `git fetch origin <base>` first, or pick a different base. Do not try to `fetch` a branch that may not exist on origin — fail loud instead.

```bash
git rev-list --left-right --count origin/<base>...HEAD
```

Returns `<behind>\t<ahead>`.

- If `behind == 0` → already up to date with `<base>`. Report and stop.
- Otherwise → show the summary (e.g. "behind `origin/dev` by 7, ahead by 3") before the next question.

### 5. Handle dirty tree

If the working tree is dirty, ask via `AskUserQuestion`:

- **Stash** → `git stash push -m "pre-sync-<base>: <branch>"`, continue, restore with `git stash pop` at the end. Warn user that `pop` may conflict after rebase/merge.
- **Commit first** → stop; point user to the `commit` skill.
- **Abort** → stop with no changes.

Never proceed with a dirty tree — both rebase and merge require a clean index.

### 6. Pick strategy (always ask)

With the divergence summary already shown, ask via `AskUserQuestion`:

| Option | Effect | When preferred |
|---|---|---|
| **Rebase** (linear history) | `git rebase origin/<base>` | Default for feature branches; keeps history clean. Requires `--force-with-lease` on next push if already pushed. |
| **Merge** (preserve history) | `git merge origin/<base>` | When the branch is already pushed and shared, or when you want to preserve the exact sequence. No force-push needed. |
| **Abort** | stop | User changed their mind. |

Warn explicitly: **if rebase is chosen and the branch was already pushed, the next push must be `git push --force-with-lease origin <branch>`**. Do not push here.

### 7. Execute

Before executing, **remember the stash name** if step 5 created one (e.g. `pre-sync-<base>: <branch>`). You will need it in the conflict path.

#### Rebase

```bash
git rebase origin/<base>
```

- On success → proceed to step 8.
- On conflict → **stop immediately**. Do NOT attempt auto-resolution. Go to step 9 (Report) with the conflict-path output; **do not skip to step 8** (stash pop cannot run mid-rebase).

#### Merge

```bash
git merge --no-ff origin/<base>     # --no-ff preserves intent; repo convention may override
```

- On success → proceed to step 8.
- On conflict → stop, go to step 9 with conflict-path output (no stash pop mid-merge).

### 8. Restore stash (if any)

If step 5 stashed changes:

```bash
git stash pop
```

- On clean pop → report.
- On pop conflict → stop, list conflicts, tell user the stash is still available (`git stash list`) and how to resolve.

### 9. Report

**Success (return to caller):** essentials only.

```text
rebased onto origin/dev (new HEAD: a1b2c3d)
next push: --force-with-lease
```

Always include the base (`origin/<base>`) — the caller needs to know which was used. For merge, drop the force-with-lease line. If stash restore had no conflict, don't mention it; if it conflicted, add `stash-pop-conflict: <files>`.

**Failure / conflict:** full context. Critical — if step 5 created a stash, **the user will forget about it** after resolving the conflict. You MUST name the stash explicitly and tell them to `git stash pop` AFTER finishing the rebase/merge.

Include:

- Operation attempted (rebase vs merge, onto which `origin/<base>`).
- Conflicting files (`git diff --name-only --diff-filter=U`).
- Current state: mid-rebase / mid-merge; HEAD sha.
- **Pending stash** (if step 5 stashed): name it explicitly (e.g. `stash@{0}: pre-sync-dev: feature/PROJ-123-foo`). Verify with `git stash list` before reporting.
- Ordered recovery commands — stash reminder always last, after the conflict is fully resolved:

  ```text
  1. Resolve conflicts in: <files>
  2. git add <files>
  3. git rebase --continue   (or git rebase --abort to revert)
  4. After the rebase finishes cleanly: git stash pop   ← do not skip; your uncommitted changes are here
  ```

  For merge: step 3 is `git commit` to accept merge message; step 4 same stash pop.

If no stash was created, omit step 4 but still order commands clearly.

## Rules

- **Always ask the base** for `feature/*` and `bugfix/*`. `hotfix/*` is strict (`main`/`master`).
- **Always ask** rebase vs merge. No silent default.
- **Never force-push** from this skill.
- **Never force-reset** the branch or base.
- **Never auto-resolve** conflicts or auto-abort.
- **Never switch branches** mid-operation — sync applies to the current branch only.
- **Never skip hooks** — respect pre-push/pre-rebase hooks.
- **Never hard-code `master`** — detect long-lived branches from `origin`.

## Examples

### Feature branch, staging flow

```text
branch: feature/PROJ-123-availability
→ detect bases: origin/main, origin/dev
→ type = feature → ask: dev / main / abort
→ user: dev
→ behind origin/dev: 7, ahead: 3
→ ask: rebase / merge / abort
→ user: rebase
→ git rebase origin/dev    ok
→ report: rebased onto origin/dev; next push needs --force-with-lease
```

### Hotfix (strict base)

```text
branch: hotfix/PROJ-900-payment-gate
→ detect bases: origin/main, origin/dev
→ type = hotfix → base = origin/main (no question)
→ behind: 3, ahead: 1
→ ask: rebase / merge / abort
→ user: merge
→ git merge --no-ff origin/main    ok
→ report: merged origin/main
```

### Legacy repo (master still in use)

```text
branch: hotfix/PROJ-900-payment-gate
→ detect bases: origin/master (no main) → legacy repo
→ type = hotfix → base = origin/master
→ …
```

### User override contradicting type

```text
branch: hotfix/PROJ-900-payment-gate
user: "sync with dev"
→ warn: hotfix should sync against main; dev is unusual
→ ask: proceed with dev / switch to main / abort
```

### Conflict

```text
→ git rebase origin/dev
CONFLICT (content): src/module/File.ext
→ stop
→ report files + instructions; do not abort
```
