---
name: create-branch
description: Create a new git branch following a standard naming convention (feature|bugfix|hotfix/<ticket-or-slug>-<short-desc>). Accepts an optional ticket ID (Jira/Linear style `[A-Z]+-\d+`, GitHub `#\d+`, or bare numeric). Ensures the base branch is up-to-date and warns about uncommitted work. Invoke when the user wants to start a new task, asks for a branch, or mentions a ticket as a starting point.
allowed-tools:
  - Bash(git status)
  - Bash(git status --porcelain*)
  - Bash(git branch)
  - Bash(git branch -a*)
  - Bash(git branch -r*)
  - Bash(git branch -v*)
  - Bash(git branch --show-current)
  - Bash(git branch --list*)
  - Bash(git fetch origin)
  - Bash(git fetch origin *)
  - Bash(git fetch --all*)
  - Bash(git checkout *)
  - Bash(git switch *)
  - Bash(git pull --ff-only*)
  - Bash(git rev-parse*)
  - Bash(git show-ref*)
  - Bash(git ls-remote*)
  - Bash(git stash push*)
  - Bash(git stash list*)
  - Bash(git stash pop*)
  - Bash(git stash show*)
  - Bash(git for-each-ref*)
  - Bash(git symbolic-ref*)
  - Bash(git log --oneline*)
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
  # out-of-scope destructive (allow has broader git branch / stash patterns)
  - Bash(git branch -d*)
  - Bash(git branch -D*)
  - Bash(git branch --delete*)
  - Bash(git stash drop*)
  - Bash(git stash clear*)
  - Bash(git commit*)
  - Bash(git merge*)
  - Bash(git rebase*)
---

# Create Branch

Create a new git branch following the standard naming convention, from an up-to-date base. **This skill is the single source of truth for branch conventions** — the git-agent and other skills defer to it.

## Branch conventions

### Long-lived branches

- **`main`** (modern default) / **`master`** (legacy repos) — production. Only stable, reviewed, approved code.
- **`dev`** / **`stage`** — staging environment where changes are tested before prod.

Modern repos ship `main`; `master` still shows up on older repos — both behave identically as the production branch. **Always detect** what exists on `origin` rather than hard-coding:

Canonical detection (use `for-each-ref`, not `git branch -r | grep` — the latter includes the `origin/HEAD -> origin/main` arrow line and leading whitespace that break the anchor):

```bash
git for-each-ref --format='%(refname:short)' 'refs/remotes/origin/*' \
  | grep -E '^origin/(main|master|dev|stage)$'
```

If both `main` and `master` exist (rare, usually mid-migration), prefer `main` and ask the user whether `master` is about to be retired.

### Working branch format

All names are **English only**:

```text
<type>/<ticket-or-slug>-<short-desc>
```

Three types:

- **`feature/`** — new functionality or larger changes.
- **`bugfix/`** — non-urgent bug fix.
- **`hotfix/`** — urgent production fix; branches from `main` / `master`.

### Ticket ID (optional)

If the project uses a ticket tracker, the `<ticket>` slot carries its ID:

- **Jira / Linear-style:** `[A-Z]+-\d+` → e.g. `PROJ-123`, `ABC-42`.
- **GitHub Issues:** `#\d+` → stored as `issue-42` in the branch (leading `#` is not legal in a ref name).
- **Bare numeric** (some legacy trackers): `\d{3,7}` → e.g. `91011`.

If there is no ticket, skip the ticket portion and use only the short description: `feature/add-product-availability`.

When extracting from user input, try Jira/Linear first (`[A-Z]+-\d+`); fall back to bare numeric only if the project's existing branches use that style. Check `git branch -a` for existing branch naming. If still ambiguous, ask.

### Short description

- Kebab-case, lowercase, 2-5 words.
- No articles, no punctuation, no diacritics.
- Describes *what*, not *how*.

### Examples

```text
feature/PROJ-123-product-availability  # Jira/Linear-style ticket
bugfix/ABC-42-price-calc               # Jira/Linear-style ticket
hotfix/PROJ-900-payment-gate           # Jira/Linear-style → branches off main
feature/add-user-auth                  # no ticket ID
feature/issue-42-dark-mode             # GitHub issue #42
feature/91011-add-auth                 # bare numeric (legacy)
```

### Base branch by type

- `feature/*`, `bugfix/*` → base is `dev` / `stage` if the project uses one; otherwise `main` (or `master` on legacy repos).
- `hotfix/*` → **always** `main` (or `master` on legacy repos).

If the user picks a base that contradicts the type (e.g. `hotfix/*` off `dev`), warn via `AskUserQuestion` and confirm before proceeding.

## Procedure

### 1. Gather inputs

Collect three pieces: **type**, **ticket (optional)**, **description**. Extract from the user's message when possible; otherwise ask via `AskUserQuestion`.

- **Type** — if missing, ask: `feature` / `bugfix` / `hotfix`.

- **Ticket** — try matching the Jira/Linear pattern `[A-Z]+-\d+` in the user's message first. If found, use it. If not found, detect which convention the project uses by counting matches in existing branches:

  ```bash
  ticket_count=$(git branch -a \
    | grep -cE '(feature|bugfix|hotfix)/[A-Z]+-[0-9]+(-|$)' || true)
  numeric_count=$(git branch -a \
    | grep -cE '(feature|bugfix|hotfix)/[0-9]{3,7}(-|$)' || true)
  no_ticket_count=$(git branch -a \
    | grep -cE '(feature|bugfix|hotfix)/[a-z]' || true)
  ```

  Note the `(-|$)` anchor on the first two — without it the regex would truncate at the first `-`, and a ticket ID like `PROJ-123` would be mistaken for only `PROJ`.

  Decide:

  - Majority have ticket IDs → ask the user for one.
  - Majority have no ticket ID → skip; use description only.
  - Mixed or empty repo → ask the user whether this project uses ticket IDs.

  **Never invent a ticket ID.** If the user doesn't supply one, omit it.

- **Description** — if missing, ask for a 2-5 word short description.

Normalize the description: lowercase, replace spaces with `-`, strip articles (`a`, `an`, `the`), strip punctuation and diacritics. Example: `"Add Product Availability"` → `add-product-availability`.

### 2. Inspect current state

```bash
git status              # never use -uall
git branch --show-current
git fetch origin        # refresh remote refs
```

Check for:

- **Uncommitted changes** (modified / staged / untracked tracked files).
- **Current branch**: is it `main` / `master`, or a working branch?
- **Base branch staleness**: is local base behind its remote?

### 3. Handle uncommitted work

If the working tree is dirty, ask via `AskUserQuestion` how to proceed:

- **Stash** — `git stash push -m "pre-branch: <target-name>"`, then continue. Tell the user how to restore (`git stash pop`).
- **Commit first** — stop this skill; direct the user to the `commit` skill, then rerun.
- **Abort** — stop without changes.

Never silently discard work.

### 4. Determine base branch

Pick by type (see "Base branch by type" in conventions above):

- `hotfix/*` → `main` / `master` **always**. If only `main` exists, use that; never branch a hotfix off `dev` / `stage`.
- `feature/*` / `bugfix/*` → prefer `dev` / `stage` if the project has one; otherwise `main` / `master`.

Detect which long-lived branches exist:

```bash
git for-each-ref --format='%(refname:short)' 'refs/remotes/origin/*' \
  | grep -E '^origin/(main|master|dev|stage)$'
```

If the user explicitly requests a different base, verify it exists on `origin`. If it contradicts the type (e.g. `hotfix` off `dev`), warn and confirm via `AskUserQuestion` before proceeding.

### 5. Sync the base

```bash
git checkout <base>
git pull --ff-only origin <base>
```

If `--ff-only` fails (local diverged), stop and report. Do not force-reset the user's base branch.

### 6. Check for name collision

```bash
git rev-parse --verify <new-branch>        # local
git ls-remote --heads origin <new-branch>  # remote
```

If the branch exists locally or remotely, ask via `AskUserQuestion`:

- **Checkout existing** — `git checkout <name>`.
- **Pick a different description** — re-prompt for description.
- **Abort**.

Never delete or overwrite an existing branch in this skill.

### 7. Create and switch

```bash
git checkout -b <type>/<ticket-or-slug>-<short-desc>
```

### 8. Report

**Success (return to caller):** only the essentials, no step narration.

```text
branch: feature/PROJ-123-availability (from main@a1b2c3d)
stashed: pre-branch-PROJ-123
```

Omit `stashed` if no stash. Nothing else — no "successfully created", no command list.

**Failure:** full context so caller can react.

- What was attempted (e.g. "create feature/PROJ-123-foo from main").
- Error (stderr snippet or which step failed: fetch / pull --ff-only / checkout -b).
- Current state: on which branch, HEAD sha, any stash created.
- 2-3 recovery options (e.g. "resolve `pull --ff-only` divergence manually; rerun; or pick a different base").

## Rules

- **Language:** branch descriptions in English.
- **Never push** the new branch here — that is for `create-pr` or explicit user request.
- **Never delete** an existing branch with a conflicting name.
- **Never reset --hard** the base branch to recover from divergence — stop and report.
- **Never skip** the uncommitted-work check.
- **Never invent a ticket ID** if the user didn't supply one and the project doesn't use them.

## Examples

### Happy path with ticket

> User: "start PROJ-123, add product availability display"

1. Extract: ticket=`PROJ-123`, description=`add product availability display`.
2. Ask for type → user picks `feature`.
3. Normalize description → `add-product-availability`.
4. Clean tree, on `main`, up-to-date.
5. Create `feature/PROJ-123-add-product-availability`.

### Happy path without ticket

> User: "new branch: refactor the auth flow"

1. Project has no ticket-style branches → skip ticket.
2. Ask for type → user picks `refactor` → not a valid type → ask again → `feature`.
3. Normalize → `refactor-auth-flow`.
4. Create `feature/refactor-auth-flow`.

### Dirty tree

> User: "new branch for ABC-42"

1. `git status` shows staged + unstaged changes.
2. Ask: stash / commit / abort → user picks stash.
3. `git stash push -m "pre-branch: bugfix/ABC-42-..."`.
4. Ask for type + description → `bugfix` + `price-calc`.
5. Sync base, create `bugfix/ABC-42-price-calc`.
6. Report: stash saved, restore with `git stash pop`.
