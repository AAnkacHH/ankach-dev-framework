---
name: create-pr
description: Open a pull request via gh CLI. Extracts any ticket ID from the branch, generates a concise title and a body with Summary + Test plan, pushes the branch with upstream tracking, and returns the PR URL. Invoke when the user wants to open a PR, ask for review, or finish a task.
allowed-tools:
  - Bash(git status)
  - Bash(git status --porcelain*)
  - Bash(git branch --show-current)
  - Bash(git branch -r*)
  - Bash(git branch -a*)
  - Bash(git fetch origin)
  - Bash(git fetch origin *)
  - Bash(git fetch --prune*)
  - Bash(git rev-list*)
  - Bash(git rev-parse*)
  - Bash(git symbolic-ref*)
  - Bash(git log --oneline*)
  - Bash(git log --format=%s*)
  - Bash(git log -1*)
  - Bash(git diff --stat*)
  - Bash(git diff --name-only*)
  - Bash(git push -u origin *)
  - Bash(git push origin *)
  - Bash(gh auth status)
  - Bash(gh repo view*)
  - Bash(gh pr create*)
  - Bash(gh pr view*)
  - Bash(gh pr list*)
  - AskUserQuestion
disallowed-tools:
  # destructive / force variants
  - Bash(git push -f*)
  - Bash(git push --force)
  - Bash(git push --force *)
  - Bash(git push * --force*)
  - Bash(git push * -f*)
  - Bash(git push *+*)
  - Bash(git push --mirror*)
  - Bash(git push * --delete*)
  - Bash(git push * :*)
  - Bash(git push origin :*)
  - Bash(git push origin --delete*)
  - Bash(git push origin +*)
  - Bash(git push * HEAD:refs/heads/main*)
  - Bash(git push * HEAD:refs/heads/master*)
  - Bash(git reset*)
  - Bash(git clean*)
  - Bash(git checkout*)
  - Bash(git switch*)
  - Bash(git commit*)
  - Bash(git merge*)
  - Bash(git rebase*)
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
  # gh dangerous (gh pr* broadly allowed via create/view/list patterns)
  - Bash(gh pr merge*)
  - Bash(gh pr close*)
  - Bash(gh pr edit*)
  - Bash(gh pr comment*)
  - Bash(gh pr review*)
  # content-leak via PR body file
  - Bash(gh pr create * --body-file*)
  - Bash(gh pr create * -F *)
  - Bash(gh pr create * --body=@*)
---

# Create Pull Request

Open a PR from the current working branch to the repository's default branch. Push the branch first if needed. Never merge or close PRs in this skill.

## Procedure

### 1. Preflight

```bash
git status                       # clean tree required; never -uall
git branch --show-current        # current branch
git fetch origin
gh auth status                   # verify gh is authenticated
```

Fail fast if:

- Not inside a repo or no `origin` remote.
- `gh` is not authenticated → tell the user to run `gh auth login` via `! gh auth login`.
- Current branch is a long-lived branch (`main`, `master` on legacy repos, `dev`, `stage`, or `release/*`) → refuse; a PR needs a working branch.
- Dirty tree → ask user to commit or stash first (point to `commit` skill). Do not silently stash.

### 2. Determine base branch

Detect the repo's default branch:

```bash
gh repo view --json defaultBranchRef -q .defaultBranchRef.name
# fallback: git symbolic-ref refs/remotes/origin/HEAD | sed 's|.*/||'
```

Use that as the PR base. If the user explicitly asks for a different base, use theirs after verifying it exists on `origin`.

### 3. Check sync state

```bash
git rev-list --left-right --count origin/<base>...HEAD
```

If behind by > 0, warn the user and ask via `AskUserQuestion`:

- **Sync first** (recommended) → stop and invoke `sync-branch` skill, then rerun this one.
- **Proceed anyway** → continue; reviewers may request a rebase.
- **Abort**.

### 4. Extract ticket ID (optional)

From current branch name, try these patterns in order:

```bash
git branch --show-current | grep -oE '[A-Z]+-[0-9]+' | head -1   # Jira/Linear
# fallback: issue-N → #N
git branch --show-current | grep -oE 'issue-[0-9]+' | head -1
```

If a ticket ID is found, include it in the title (step 5). **If not found, skip it — do not fabricate one**, do not ask unless the user's message mentions a tracker.

### 5. Draft PR title

With ticket ID:

```
TICKET-ID Short description
```

Without ticket ID:

```
Short description
```

Rules:

- Under 70 chars total.
- Capitalize description; no trailing period.
- Imperative mood (`Add`, `Fix`, `Refactor`), matching the dominant commit type on the branch.
- Derive from the branch description and/or the single commit subject if there's only one commit.

Examples:

- `PROJ-123 Add product availability display`
- `Fix race condition in token refresh`

If the branch has multiple commits touching different concerns, ask the user to confirm the title.

### 6. Draft PR body

Structure:

```markdown
## Summary
- <1-3 bullets: what this PR does, at a high level>
- <focus on the "why", not line-by-line "what">

## Test plan
- [ ] <concrete check #1>
- [ ] <concrete check #2>
- [ ] <concrete check #3>

Related: TICKET-ID
```

How to fill:

- Read `git log origin/<base>..HEAD --oneline` and the diff summary (`git diff origin/<base>...HEAD --stat`) to summarize.
- Test plan items should be concrete: a command to run, a URL to click, a specific scenario — not "tests pass".
- If the PR closes the ticket, add `Closes: TICKET-ID` (or `Closes #42` for GitHub Issues).
- Drop the `Related:` line if there's no ticket.

Ask the user via `AskUserQuestion` before opening if anything is unclear or if the diff is very large (> 500 lines).

### 7. Push branch

Before pushing, detect whether the remote branch already exists and is ahead of local:

```bash
# detect if the remote branch exists and is ahead of local
if git rev-parse --verify origin/<current-branch> >/dev/null 2>&1; then
  counts=$(git rev-list --left-right --count origin/<current-branch>...HEAD)
  remote_ahead=$(echo "$counts" | awk '{print $1}')
  if [ "$remote_ahead" -gt 0 ]; then
    # remote is ahead — stop and report; do not force-push from this skill
    exit 1
  fi
fi
```

If the remote branch exists and is ahead (`remote_ahead > 0`), stop immediately. Someone else pushed commits you don't have. Tell the user to investigate (`git fetch origin <branch> && git log HEAD..origin/<branch>`) and resolve manually. Do not force-push here.

Otherwise push:

```bash
git push -u origin <current-branch>
```

- If the remote branch already exists and is ahead of local → stop. Do not force-push from this skill; tell the user to investigate.
- If a rebase happened earlier and the remote diverged → warn; require explicit user confirmation before running `git push --force-with-lease origin <current-branch>`. Default to stopping.

### 8. Create the PR

Canonical form (prefer `--body-file -` on stdin with a quoted-delimiter heredoc):

```bash
gh pr create \
  --base <base> \
  --head <current-branch> \
  --title "TICKET-ID Short description" \
  --body-file - <<'EOF'
## Summary
- ...

## Test plan
- [ ] ...

Related: TICKET-ID
EOF
```

Alternative (captured via command substitution):

```bash
gh pr create \
  --base <base> \
  --head <current-branch> \
  --title "TICKET-ID Short description" \
  --body "$(cat <<'EOF'
## Summary
- ...

## Test plan
- [ ] ...

Related: TICKET-ID
EOF
)"
```

> **Body quoting is mandatory.** Always build the body via a quoted-delimiter heredoc (`<<'EOF' ... EOF`) piped into `gh pr create --body-file -` or captured via `"$(cat <<'EOF' ... EOF)"`. The single-quoted delimiter prevents `$(...)`, backticks, and `$var` in commit subjects (spliced from `git log`) from being executed. Never build the body with a double-quoted string that interpolates commit content.

Flags to consider (ask if unclear):

- `--draft` when the user says "draft", "WIP", or explicitly requests it.
- `--reviewer <user>` when reviewers are named.
- `--label <label>` when the repo has required labels.
- `--assignee @me` by default.

### 9. Report

**Success:** one line.

```text
PR: https://github.com/org/repo/pull/123 (draft, base=main, ticket=PROJ-123)
```

Omit `draft` if it is a normal PR, omit `ticket=` if there is none. That's the whole output on success.

**Failure:** full context.

- Step that failed: preflight / push / `gh pr create`.
- Error message (stderr from `gh` or `git push`).
- Repo state: branch, HEAD sha, whether branch was pushed, auth status.
- Recovery: e.g. "rerun `gh auth login`; or push manually with `git push -u origin <branch>`; or pick a different base".

## Rules

- **Never push to the prod branch (`main` / `master`), `dev` / `stage`, or `release/*`.**
- **Never `--force-push`** without explicit user confirmation, and never to protected branches.
- **Never merge or close** PRs in this skill.
- **Never invent** a ticket ID if the branch lacks one — omit it.
- **Never include secrets** in the PR body (API keys, tokens). Scan the diff summary before posting.
- **Language:** PR title and body in English.

## Examples

### Happy path

> User: "open a PR for this branch"

1. Branch `feature/PROJ-123-product-availability`, clean tree, up to date with `origin/main`.
2. Ticket: `PROJ-123`.
3. Title: `PROJ-123 Add product availability display`.
4. Body: summary from 3 commits + test plan from diff.
5. `git push -u origin feature/PROJ-123-product-availability`.
6. `gh pr create --base main --head feature/PROJ-123-product-availability ...`.
7. Report URL.

### Behind base

> Branch behind by 5 commits relative to the chosen base.

→ Ask: sync first / proceed / abort. If sync → stop here and recommend `sync-branch` skill (which picks the right base for the branch type).
