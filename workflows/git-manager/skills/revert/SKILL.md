---
name: revert
description: Create a revert commit that inverts one or more existing commits. Additive (new commit, no history rewrite). Handles single commit, range, and merge-commit revert. Enforces a working branch (never reverts directly on main/master/dev/stage). Invoke when the user wants to undo a commit that has already been merged / pushed, "roll back", or "revert PR".
allowed-tools:
  - Bash(git status)
  - Bash(git status --porcelain*)
  - Bash(git branch --show-current)
  - Bash(git log --oneline*)
  - Bash(git log -n *)
  - Bash(git log --format=*)
  - Bash(git log -1*)
  - Bash(git show --stat*)
  - Bash(git show --name-only*)
  - Bash(git rev-parse*)
  - Bash(git cat-file -t*)
  - Bash(git cat-file -e*)
  - Bash(git revert *)
  - Bash(git diff --name-only*)
  - Bash(git diff --stat*)
  - Skill
  - AskUserQuestion
disallowed-tools:
  # commit is out of scope for this skill (revert only — hand off to commit skill)
  - Bash(git commit*)
  # cat-file -p leaks content (narrow -t / -e are allowed)
  - Bash(git cat-file -p*)
  # dangerous destructive ops (belt-and-suspenders)
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

# Revert Commit(s)

Create a new commit (or commits) that invert the changes of existing commits. This is **additive** — no history rewrite, safe on shared branches. Secret-file patterns come from `../_shared/secret-patterns.md`.

## When to use vs not

- **Use `revert`** when the bad commit is already on a shared branch (merged to `main`/`dev`, pushed, or seen by others). Creates a clean audit trail: the new commit documents the reversal.
- **Do NOT use `revert`** when the commit is local, unpushed, and you own the branch — `git reset --hard <sha>^` is simpler there, but that's outside this skill's scope (and the harness blocks it anyway).

## Revert shapes

### Single commit

```bash
git revert <sha>
```

Creates one new commit inverting `<sha>`.

### Range

```bash
git revert <older-sha>..<newer-sha>       # reverts each commit in the range in reverse order
git revert --no-commit <sha1> <sha2>      # squash multiple reverts into one commit (commit manually after)
```

Use `--no-commit` + manual commit when several related reverts should land as a single atomic change in the log.

### Merge commit

```bash
git revert -m 1 <merge-sha>
```

`-m 1` picks the **mainline parent** (usually the branch that received the merge — e.g. `main`). Use `-m 2` only when reverting a merge that came in from a feature branch perspective. Always confirm mainline with the user if unsure.

## Procedure

### 1. Preflight

```bash
git status
git branch --show-current
```

Stop if:

- Current branch is a long-lived branch (`main` / `master` / `dev` / `stage` / `release/*`) → refuse; require a working branch. Suggest invoking the `create-branch` skill first (e.g. `hotfix/revert-availability`).
- Working tree is dirty → ask via `AskUserQuestion`: **stash** (invoke `stash-manager`), **commit first**, or **abort**. Never revert on a dirty tree.

### 2. Identify target commit(s)

Extract from the user's request. Accept:

- Full or short SHA (`a1b2c3d`).
- `HEAD~N` / `HEAD^` references.
- PR number → resolve via `gh pr view <N> --json mergeCommit -q .mergeCommit.oid` (requires `gh`) — but the `gh` tool is outside this skill's allowlist; ask the user for the SHA instead, or suggest invoking `search-commits` to find it.

Verify each target exists:

```bash
git cat-file -e <sha>^{commit}     # exit 0 = exists
git log --format='%h %s' -1 <sha>  # show subject for confirmation
```

Display subject + date + author (no diff content) and ask the user to confirm: **"Revert `a1b2c3d Feat(api): PROJ-123 Add availability` by Alice (2025-03-14)?"**

If the commit is a merge commit (`git cat-file -t <sha>` → commit with multiple parents), ask which parent is mainline and use `-m <N>` accordingly.

**SHA validation (mandatory before splicing).** Before passing `<sha>` to any git command, verify it matches `^[A-Fa-f0-9]{4,40}$` or a safe short-ref form (`HEAD`, `HEAD~\d+`, `HEAD\^+`). Reject anything else — a value like `a1b2c3d; rm -rf ~` would satisfy the `Bash(git revert *)` allowlist but break out of the command. Use:

```bash
case "$sha" in
  [A-Fa-f0-9]*) : ;;
  HEAD|HEAD~*|HEAD^*) : ;;
  *) echo "unsafe SHA: $sha"; exit 1 ;;
esac
```

Same rule for the `-m <N>` parent number: must match `^[1-9]$`. Reject any other value before splicing into the `git revert -m <N> <sha>` command.

### 3. Run the revert

```bash
git revert <sha>                      # single
git revert <older>..<newer>           # range
git revert -m <parent-num> <sha>      # merge
```

- On success → git opens an editor with a default message. The skill's allowlist does not include `$EDITOR`; use the default subject (`Revert "<original-subject>"`). To customize, cancel and rerun with `--no-commit`, edit the index, then delegate the final commit to the `commit` skill.
- On conflict → **stop immediately**. Report conflicting files and instruct the user: resolve → `git add <files>` → `git revert --continue` (or `git revert --abort` to revert the revert). Never auto-abort.

### 4. Verify

```bash
git log --oneline -3
```

Confirm the new revert commit appears on top.

### 5. Hand off

The revert commit is local. Tell the user the next step:

- Push + open PR → delegate to `create-pr` skill.
- Or direct push to a working branch → the user must do it explicitly (skill does not push).

## Commit message

Git's default revert subject is `Revert "<original-subject>"`. This is acceptable for traceability. **Recommend** (via `AskUserQuestion`):

- **Keep default** — clean audit trail, no rewrite needed.
- **Reformat to project convention** — use the `commit` skill after `git revert --no-commit` to produce a message like:

  ```text
  Revert(api): PROJ-900 Revert broken availability endpoint

  - Reverts a1b2c3d (PROJ-123 Add product availability endpoint)
  - Reason: regression in downstream consumer.

  Co-Authored-By: {{MODEL}} <noreply@anthropic.com>
  ```

`Revert` is added as a valid commit type for this purpose; the rest follows `commit` skill rules (scope, ticket, trailer — substitute `{{MODEL}}`).

## Rules

- **Working branch only** — refuse on `main` / `master` / `dev` / `stage` / `release/*`.
- **Never auto-resolve** revert conflicts.
- **Never push** — the skill produces a commit; push via `create-pr` or explicit user action.
- **Never inspect commit contents via `-p`** — path-level identity (`--stat`, `--name-only`) is enough to confirm the target.
- **Merge-commit revert requires explicit `-m N`** — ask the user which parent is mainline; do not guess.
- **Show subject + author + date before reverting**, never silently act on a bare SHA.

## Examples

### Single commit revert on a new branch

```text
user: "revert PROJ-123"
→ search commits for PROJ-123 → sha a1b2c3d
→ current branch: main → refuse
→ invoke create-branch: hotfix/PROJ-900-revert-availability
→ confirm: "Revert 'Feat(api): PROJ-123 Add availability' by Alice (2025-03-14)?"
→ user: yes
→ git revert a1b2c3d    ok
→ report: revert commit c3d4e5f on hotfix/PROJ-900-revert-availability
→ next: invoke create-pr
```

### Merge-commit revert

```text
user: "revert the PR for PROJ-123"
→ find merge sha: m1m2m3d (2 parents: main, feature/PROJ-123)
→ ask: which parent is mainline? main / feature/PROJ-123
→ user: main (the one the merge landed on)
→ git revert -m 1 m1m2m3d    ok
→ report: revert commit on hotfix/PROJ-900-revert-availability
```

### Conflict

```text
→ git revert a1b2c3d
CONFLICT (content): src/module/File.ext
→ stop
→ report files; instruct: resolve, git add, git revert --continue (or --abort)
```

## Report

**Success (return to caller):** essentials.

```text
reverted a1b2c3d on hotfix/PROJ-900-revert-availability (new HEAD: c3d4e5f)
next: push + PR (invoke create-pr)
```

For a range, include count:

```text
reverted 3 commits (a1..c3d) on hotfix/PROJ-900-revert-range (new HEAD: f7g8h9i)
```

**Failure:** full context — operation attempted, conflicting files if any, current state (branch, HEAD, whether a revert is mid-flight), recovery commands (`git revert --continue` / `--abort`).
