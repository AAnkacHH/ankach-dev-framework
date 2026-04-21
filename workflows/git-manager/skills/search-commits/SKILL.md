---
name: search-commits
description: Universal git history and commit search. Finds commits, files, authors, or code changes using pickaxe (-S), regex (-G), message grep, author, date, path, blame, line-range history, git show, and GitHub commit/PR search. Invoke when the user asks "who/when/where/why" about commits, file history, deleted code, or past changes. Read-only — never modifies the repository.
allowed-tools:
  - Bash(git log)
  - Bash(git log --oneline*)
  - Bash(git log --all*)
  - Bash(git log --format=*)
  - Bash(git log --pretty=*)
  - Bash(git log --grep=*)
  - Bash(git log --author=*)
  - Bash(git log --committer=*)
  - Bash(git log --since=*)
  - Bash(git log --until=*)
  - Bash(git log --before=*)
  - Bash(git log --after=*)
  - Bash(git log -n *)
  - Bash(git log -1*)
  - Bash(git log -5*)
  - Bash(git log -10*)
  - Bash(git log -20*)
  - Bash(git log -50*)
  - Bash(git log -S *)
  - Bash(git log -G *)
  - Bash(git log -L *)
  - Bash(git log --follow*)
  - Bash(git log --diff-filter=*)
  - Bash(git log --no-merges*)
  - Bash(git log --merges*)
  - Bash(git log --invert-grep*)
  - Bash(git log --left-right*)
  - Bash(git log --cherry-mark*)
  - Bash(git log --stat*)
  - Bash(git log --name-only*)
  - Bash(git log --name-status*)
  - Bash(git log -i *)
  - Bash(git log --all --grep=*)
  - Bash(git log --all --author=*)
  - Bash(git log --all -S *)
  - Bash(git log --all -G *)
  - Bash(git log --all --follow*)
  - Bash(git log --all --oneline*)
  - Bash(git log --all --diff-filter=*)
  - Bash(git show --stat*)
  - Bash(git show --name-only*)
  - Bash(git show --name-status*)
  - Bash(git show --format=*)
  - Bash(git show --pretty=*)
  - Bash(git show --no-patch*)
  - Bash(git show -s *)
  - Bash(git blame -L *)
  - Bash(git blame --ignore-rev *)
  - Bash(git blame -w *)
  - Bash(git blame -M *)
  - Bash(git blame -C *)
  - Bash(git diff --stat*)
  - Bash(git diff --name-only*)
  - Bash(git diff --name-status*)
  - Bash(git diff --shortstat*)
  - Bash(git diff --numstat*)
  - Bash(git diff origin/*)
  - Bash(git rev-list*)
  - Bash(git rev-parse*)
  - Bash(git shortlog*)
  - Bash(git reflog)
  - Bash(git reflog show*)
  - Bash(git branch)
  - Bash(git branch -a*)
  - Bash(git branch -r*)
  - Bash(git branch --contains*)
  - Bash(git branch --merged*)
  - Bash(git branch --no-merged*)
  - Bash(git tag)
  - Bash(git tag -l*)
  - Bash(git tag --contains*)
  - Bash(git describe*)
  - Bash(git for-each-ref*)
  - Bash(git name-rev*)
  - Bash(git cherry*)
  - Bash(git status)
  - Bash(gh search commits*)
  - Bash(gh search prs*)
  - Bash(gh search issues*)
  - Bash(gh pr list*)
  - Bash(gh pr view *)
  - Bash(gh api repos/*/commits*)
  - Bash(gh api repos/*/pulls*)
  - Bash(gh api repos/*/issues*)
  - AskUserQuestion
disallowed-tools:
  # destructive
  - Bash(git commit*)
  - Bash(git push*)
  - Bash(git reset*)
  - Bash(git checkout*)
  - Bash(git switch*)
  - Bash(git merge*)
  - Bash(git rebase*)
  - Bash(git clean*)
  - Bash(git branch -d*)
  - Bash(git branch -D*)
  - Bash(git branch --delete*)
  - Bash(git update-ref*)
  - Bash(git filter-branch*)
  - Bash(git filter-repo*)
  - Bash(git replace*)
  - Bash(git gc*)
  - Bash(git reflog expire*)
  - Bash(git stash*)
  - Bash(git config*)
  - Bash(git remote add*)
  - Bash(git remote remove*)
  - Bash(git remote rm*)
  - Bash(git remote set-url*)
  # content extraction bypasses
  - Bash(git cat-file*)
  - Bash(git archive*)
  - Bash(git bundle*)
  - Bash(git grep*)
  - Bash(git fsck*)
  # gh dangerous (content leak / write)
  - Bash(gh api *contents*)
  - Bash(gh api *blobs*)
  - Bash(gh api *trees*)
  - Bash(gh api * -X POST*)
  - Bash(gh api * -X PUT*)
  - Bash(gh api * -X PATCH*)
  - Bash(gh api * -X DELETE*)
  - Bash(gh api * --method POST*)
  - Bash(gh api * --method PUT*)
  - Bash(gh api * --method PATCH*)
  - Bash(gh api * --method DELETE*)
  - Bash(gh auth login*)
  - Bash(gh auth logout*)
  - Bash(gh auth refresh*)
  - Bash(gh auth token*)
  - Bash(gh secret*)
  - Bash(gh ssh-key*)
  - Bash(gh gpg-key*)
  - Bash(gh repo delete*)
  - Bash(gh pr merge*)
  - Bash(gh pr close*)
  - Bash(gh pr edit*)
  - Bash(gh release create*)
  - Bash(gh release delete*)
  - Bash(gh workflow run*)
  - Bash(gh run rerun*)
---

# Search Commits and History

Universal read-only search across commit history, code changes, authors, files, PRs, and GitHub metadata. Designed to be extended — add new patterns to the toolbox as they appear.

## Core principle

**The user's intent drives the tool choice, not a fixed menu.** Read the question, map it to the best primitive (or combination) from the toolbox below, run it, present the findings, and let the user drill deeper.

If the intent is ambiguous, ask one clarifying question via `AskUserQuestion` — do not fire many queries hoping one lands.

## Toolbox (extensible)

### Message search

```bash
git log --all --grep='<pattern>' --oneline              # commits whose message matches
git log --all --grep='PROJ-123' --oneline               # ticket ID
git log --all -i --grep='fix.*price'                    # case-insensitive regex
git log --all --grep='typo' --invert-grep --oneline     # exclude
```

Use for: "commits mentioning PROJ-123", "all fixes about price", "what did we say about this migration".

### Code change search (pickaxe)

```bash
git log --all -S '<literal string>' --oneline           # commits that add/remove occurrences of literal
git log --all -S 'functionName' --diff-filter=D         # commits that REMOVED occurrences
git log --all -S 'functionName' --diff-filter=A         # commits that ADDED
git log --all -G '<regex>' --oneline                    # commits whose diff matches regex
```

Use for: "when was functionX deleted", "who introduced that constant", "all diffs touching a pattern". `-S` is literal + faster; `-G` is regex + catches modifications within a line.

### File / path history

```bash
git log --all --oneline -- <path>                       # history of a file/dir
git log --all --follow --oneline -- <path>              # follow renames (file only)
git log --all -p -- <path>                              # with diffs
git log --all --diff-filter=D -- <path>                 # commits that deleted the path
```

### Line / function history

```bash
git log -L <start>,<end>:<file>                         # history of a line range
git log -L :<funcName>:<file>                           # history of a function (language-aware)
git blame -L <start>,<end> -- <file>                    # per-line attribution
git blame -L <start>,<end> --ignore-rev <sha> -- <file> # skip a noisy commit (e.g. mass reformat)
```

Use for: "who wrote this line", "history of this function", "when did this block change".

### Author / date filters (stackable)

```bash
git log --all --author='<name-or-email>' --oneline
git log --all --since='2 weeks ago' --until='yesterday' --oneline
git log --all --since='2025-03-01' --committer='<email>' --oneline
git shortlog -sne --since='last month'                  # commit counts per author
```

Stack these with `--grep`, `-S`, `-G`, or `-- <path>` for precise queries.

### Specific commit inspection

```bash
git show <sha>                                          # full diff + message
git show <sha> -- <path>                                # only a path
git show <sha>:<path>                                   # file contents at that commit
git describe --contains <sha>                           # which tag/release first contains this commit
git branch --contains <sha>                             # which branches contain it
git tag --contains <sha>                                # which tags contain it
git name-rev <sha>                                      # human-readable name for a sha
```

### Divergence / comparison

```bash
git log origin/main..HEAD --oneline                     # commits only on current branch
git log HEAD..origin/main --oneline                     # commits only on main
git log --left-right --cherry-mark origin/main...HEAD   # side-by-side with cherry marks
git cherry -v origin/main                               # commits not yet upstream
git diff origin/main...HEAD --stat                      # file-level diff from merge base
```

### Lost commits / reflog

```bash
git reflog                                              # local HEAD history
git reflog show <branch>                                # per-branch reflog
git fsck --lost-found --no-reflogs                      # dangling commits
```

Use for: "I dropped a commit", "recover after rebase gone wrong".

### GitHub-wide search (cross-branch, cross-fork)

```bash
gh search commits '<query>' --repo <owner/repo>
gh search commits 'PROJ-123 author:@me' --repo <owner/repo>
gh search prs 'PROJ-123 is:merged' --repo <owner/repo>
gh pr list --search 'is:merged PROJ-123' --json number,title,url,mergedAt
gh pr view <number> --json commits,files,comments
gh api repos/<owner>/<repo>/commits?path=<path>         # paginated commit list via API
```

Use when the commit may not be local, when you need PR metadata, or when the user mentions another branch/fork.

## Workflow

### 1. Classify the intent

Quickly map the question to one or more toolbox categories:

| User question | Primary tools |
|---|---|
| "when was X removed?" | pickaxe `-S` + `--diff-filter=D` |
| "who introduced this pattern?" | pickaxe `-S` + `--diff-filter=A` or `-G` + blame |
| "all commits for PROJ-123" | `--grep='PROJ-123' --all` + `gh pr list --search` |
| "history of file F" | `git log --follow -- F` |
| "history of function X in F" | `git log -L :X:F` |
| "what changed last week by Alice" | `--since --author` |
| "which release contains commit S?" | `git describe --contains` + `git branch --contains` |
| "recover a lost commit" | `git reflog`, `fsck --lost-found` |
| "diff between my branch and main" | `git diff origin/main...HEAD --stat` |

If it fits none cleanly, ask a one-line clarifier.

### 2. Start narrow, widen if empty

- Prefer `--all` for pickaxe and grep (searches all refs).
- Cap output with `--oneline -n 50` on first pass, widen on request.
- If zero results with `-S`, try `-G` (regex). If still empty, broaden the term.

### 3. Always show, then drill

Present a compact table first — sha, date, author, subject. Then offer to drill:

- `git show <sha>` for full diff
- `gh pr view <N>` if the commit is in a PR
- Follow a file with `git log --follow -- <path>`

Do not dump 500-commit logs. If the result set is large, ask whether to filter by date / author / path.

### 4. Combine when helpful

Common combos:

- Ticket + PR: `git log --grep='PROJ-123'` **and** `gh pr list --search 'PROJ-123'` — local + remote metadata.
- Blame + ignore refactor: `git blame --ignore-rev <refactor-sha> -- <file>`.
- Pickaxe + author: `git log -S '<term>' --author='<email>'`.

### 5. Report

**Success:** the command used + compact results table. Caller may drill with another call.

```text
cmd: git log --all -S 'calculateBundlePrice' --diff-filter=D --oneline
a1b2c3d  2025-03-14  alice  Refactor(core): PROJ-650 Drop legacy bundle price
```

For multi-part results, use short sections ("Commits", "PRs"). Cap at 20 rows by default — if more exist, say `…+47 more, narrow by --since / --author / -- <path>`.

If zero results: one line — `no matches for <query> (tried: <command>)`.

**Failure:** full context.

- Command that failed + stderr.
- Likely cause (bad regex, ref not fetched, `gh` not authed).
- Next action (refetch, re-auth, rephrase query).

## Rules

- **Read-only.** No `checkout`, `merge`, `rebase`, `reset`, `push`, `commit`, `branch -d/-D`.
- **No fabrication.** If a commit/PR isn't found, say so — do not invent SHAs or numbers.
- **Prefer `--all`** for message / pickaxe searches unless the user scopes to a branch.
- **Quote carefully.** Commit messages can contain special chars; use single quotes for patterns.
- **Budget output.** Default `-n 50`; widen only on request.

### Secret safety

Git history leaks as readily as the working tree. The canonical list of secret file patterns lives in `../_shared/secret-patterns.md`; treat anything matching (or resembling) those as a secret, including filenames not explicitly listed.

- **Never `git show <sha>:<secret-path>`** — retrieves file contents at that commit.
- **Never `git blame <secret-path>`** — each line's content is shown.
- **Never `git log -p -- <secret-path>`** — full diffs leak content.
- **Never `git log -S '<literal value of a secret>'`** — the secret appears in your command history.
- **Allowed, still useful:**
  - `git log -- <secret-path>` (no `-p`) — shows commits that touched the file, without contents.
  - `git log --diff-filter=A -- <secret-path>` — when was it added.
  - `git log --diff-filter=D -- <secret-path>` — when was it removed.
  - These help confirm a leak exists without exposing the value.
- **If you suspect a secret is in history**, do not fetch its content. Report the SHA and path; recommend history scrub (`git filter-repo` / BFG) and rotation of the exposed credential. Rotation is usually more important than scrubbing — a public commit is considered compromised regardless of later rewrites.
- **Never echo retrieved content** that may be a secret into your output or the caller's context.

## Extending this skill

When a new recurring search pattern appears, add it to the Toolbox as a new subsection with:

1. The command(s).
2. One-line "use for" description.
3. An example if the flag combo is non-obvious.

Keep the Workflow section stable — it's the dispatch logic. The Toolbox is where the skill grows.

## Examples

### "When was `calculateBundlePrice` removed?"

```bash
git log --all -S 'calculateBundlePrice' --diff-filter=D --oneline
# pick the sha → inspect
git show <sha> -- src/core/
```

### "All commits and PRs for PROJ-123"

```bash
git log --all --grep='PROJ-123' --oneline
gh pr list --search 'PROJ-123' --state all --json number,title,url,mergedAt
```

### "Who last touched lines 40–60 of File.ext, ignoring the 2025 reformat"

```bash
git log --oneline -- src/module/File.ext | head -5       # find the reformat sha
git blame -L 40,60 --ignore-rev <reformat-sha> -- src/module/File.ext
```

### "What did Alice change in the last two weeks under src/api/?"

```bash
git log --all --author='alice@' --since='2 weeks ago' --oneline -- src/api/
```
