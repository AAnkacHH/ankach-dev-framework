---
name: commit
description: Create git commits following a conventional-commit format with optional ticket prefix, proper type/scope, and Co-Authored-By trailer. Invoke when the user asks to commit, save changes, or create a commit.
allowed-tools:
  - Bash(git status)
  - Bash(git status --porcelain*)
  - Bash(git diff)
  - Bash(git diff --stat*)
  - Bash(git diff --cached)
  - Bash(git diff --cached --stat*)
  - Bash(git diff --name-only*)
  - Bash(git diff --cached --name-only*)
  - Bash(git log --oneline*)
  - Bash(git log -5*)
  - Bash(git log -n *)
  - Bash(git branch --show-current)
  - Bash(git add *)
  - Bash(git commit)
  - Bash(git commit -F -*)
  - Bash(git check-ignore*)
  - Bash(git ls-files*)
  - Bash(git rev-parse*)
  - Bash(git restore --staged *)
  - AskUserQuestion
  - Read
  - Edit
disallowed-tools:
  # destructive / scope-creep git (hard-deny the ones adjacent to allowed git ops)
  - Bash(git update-ref*)
  - Bash(git filter-branch*)
  - Bash(git filter-repo*)
  - Bash(git replace*)
  - Bash(git gc*)
  - Bash(git config*)
  - Bash(git remote add*)
  - Bash(git remote remove*)
  - Bash(git remote rm*)
  - Bash(git remote set-url*)
  # never stage broad patterns
  - Bash(git add -A*)
  - Bash(git add --all*)
  - Bash(git add .)
  - Bash(git add --no-pager*)
  # never stage secrets
  - Bash(git add *.env)
  - Bash(git add *.env.*)
  - Bash(git add *.env*)
  - Bash(git add **/.env*)
  - Bash(git add .env)
  - Bash(git add .env.*)
  - Bash(git add .env*)
  - Bash(git add *.envrc*)
  - Bash(git add .envrc)
  - Bash(git add *.env.vault*)
  - Bash(git add *auth.json*)
  - Bash(git add *credentials*)
  - Bash(git add *credentials.json*)
  - Bash(git add *secrets.*)
  - Bash(git add *secrets.yml*)
  - Bash(git add *secrets.yaml*)
  - Bash(git add *secrets.json*)
  - Bash(git add *.token*)
  - Bash(git add *token*)
  - Bash(git add *.tfvars*)
  - Bash(git add *terraform.tfstate*)
  - Bash(git add *wp-config.php*)
  - Bash(git add *config/master.key*)
  - Bash(git add *config/credentials/*.key*)
  - Bash(git add *appsettings.*.json*)
  - Bash(git add *local.settings.json*)
  - Bash(git add *.pem*)
  - Bash(git add *.key*)
  - Bash(git add *.p12*)
  - Bash(git add *.pfx*)
  - Bash(git add *.pkcs12*)
  - Bash(git add *.kdbx*)
  - Bash(git add *.jks*)
  - Bash(git add *.keystore*)
  - Bash(git add *.asc*)
  - Bash(git add *.gpg*)
  - Bash(git add *firebase-adminsdk*.json*)
  - Bash(git add *service-account*.json*)
  - Bash(git add *id_rsa*)
  - Bash(git add *id_ed25519*)
  - Bash(git add *id_ecdsa*)
  - Bash(git add *id_dsa*)
  - Bash(git add *.ssh/*)
  - Bash(git add **/.ssh/**)
  - Bash(git add *.netrc*)
  - Bash(git add *.git-credentials*)
  - Bash(git add *.npmrc*)
  - Bash(git add *.yarnrc*)
  - Bash(git add *.pypirc*)
  - Bash(git add *.cargo/credentials*)
  - Bash(git add *.gradle/gradle.properties*)
  - Bash(git add *.aws/*)
  - Bash(git add **/.aws/**)
  - Bash(git add *.kube/*)
  - Bash(git add **/.kube/**)
  - Bash(git add *.docker/config.json*)
  - Bash(git add *gcloud/*)
  - Bash(git add **/gcloud/**)
  # never read secret files
  - Read(**/.env)
  - Read(**/.env.*)
  - Read(**/*.env)
  - Read(**/.envrc)
  - Read(**/.env.vault)
  - Read(**/auth.json)
  - Read(**/credentials.json)
  - Read(**/credentials)
  - Read(**/secrets.yml)
  - Read(**/secrets.yaml)
  - Read(**/secrets.json)
  - Read(**/*.token)
  - Read(**/*.tfvars)
  - Read(**/terraform.tfstate*)
  - Read(**/wp-config.php)
  - Read(**/config/master.key)
  - Read(**/config/credentials/*.key)
  - Read(**/appsettings.*.json)
  - Read(**/local.settings.json)
  - Read(**/*.pem)
  - Read(**/*.key)
  - Read(**/*.p12)
  - Read(**/*.pfx)
  - Read(**/*.pkcs12)
  - Read(**/*.kdbx)
  - Read(**/*.jks)
  - Read(**/*.keystore)
  - Read(**/*.asc)
  - Read(**/*.gpg)
  - Read(**/firebase-adminsdk*.json)
  - Read(**/service-account*.json)
  - Read(**/id_rsa*)
  - Read(**/id_ed25519*)
  - Read(**/id_ecdsa*)
  - Read(**/id_dsa*)
  - Read(**/.ssh/**)
  - Read(**/.netrc)
  - Read(**/.git-credentials)
  - Read(**/.npmrc)
  - Read(**/.yarnrc)
  - Read(**/.pypirc)
  - Read(**/.cargo/credentials*)
  - Read(**/.gradle/gradle.properties)
  - Read(**/.aws/credentials)
  - Read(**/.aws/config)
  - Read(**/.aws/**)
  - Read(**/.docker/config.json)
  - Read(**/.kube/config)
  - Read(**/.kube/**)
  - Read(**/gcloud/**)
---

# Create Git Commit

Create one or more focused git commits following a conventional-commit layout.

## Procedure

### 1. Understand current changes

Run these commands to understand the state:

```bash
git status          # never use -uall flag
git diff --stat     # overview of changed files
git diff            # full diff of unstaged changes
git diff --cached   # full diff of staged changes
git log --oneline -5  # recent commits for style reference
```

### 1b. Secret-file safety check

Before staging anything, verify that secret-file patterns are gitignored, and that no secret file appears in the changes you are about to commit.

**Patterns to check:** see the shared reference `../_shared/secret-patterns.md`. That document is the single source of truth — consult it, do not re-enumerate patterns here. Additionally, treat any filename that *looks* like it might contain credentials as a secret even if it is not in the list.

#### Step A — scan what's about to change

```bash
git status --porcelain
git diff --cached --name-only
git diff --name-only
```

If any listed or untracked file matches a secret pattern, stop immediately and warn. Do NOT stage it. Ask the user via `AskUserQuestion`: **skip this file**, **remove from the set** (if already staged: `git restore --staged <file>`), or **abort commit**.

#### Step B — verify each secret file is not tracked, then check ignore coverage

**Order matters.** For a file already committed earlier, `git check-ignore` returns exit 0 (ignored) even though the tracked content is still in history. Check tracked-state first.

For each secret pattern whose matching file exists in the working tree:

1. **Tracked check (first):**

   ```bash
   git ls-files --error-unmatch <file>
   ```

   - Exit 0 → file is **already tracked**. **Stop and warn loudly** — the secret is in git history (even if `.gitignore` mentions it now). Tell the user:
     1. Rotate the exposed credential immediately (primary action — history rewrite is secondary).
     2. `git rm --cached <file>` to untrack going forward.
     3. Scrub history with `git filter-repo` / BFG if the repo hasn't been widely shared.
     Do not attempt history rewrite in this skill.
   - Exit 1 → file is untracked. Proceed to the ignore check.

2. **Ignore check (second):**

   ```bash
   git check-ignore -v <file>
   ```

   - Exit 0 → ignored. Good, skip.
   - Exit 1 → NOT ignored. Go to Step C.

#### Step C — add missing patterns to .gitignore

If any secret file exists but is not ignored, append the pattern(s) to `.gitignore`:

1. Read current `.gitignore` (create if missing).
2. For each missing pattern that corresponds to a present file, append it on a new line under a `# Secrets` header (add the header only once).
3. Confirm via `AskUserQuestion` before committing the `.gitignore` change. Options:
   - **Commit .gitignore first, then my changes** (recommended — two commits, clean history).
   - **Bundle .gitignore into the same commit**.
   - **Just update .gitignore, don't proceed with the main commit yet**.
4. Verify with `git check-ignore -v <file>` that the pattern now matches.

Never read the contents of a secret file to "check" it — the `disallowed-tools` list blocks that. Rely on path-based checks only.

### 2. Extract ticket ID (optional)

Run `git branch --show-current` to get the current branch name.

Try to extract a ticket ID from the branch in this order:

1. Jira / Linear-style: `[A-Z]+-\d+` (e.g. `PROJ-123`, `ABC-42`)
2. GitHub-issue-style: `#\d+` (e.g. `#123`)
3. Bare numeric (only if project convention uses it): `\d{3,7}`

**If a ticket ID is found**, include it in the subject line (see step 5).
**If no ticket ID is found**, skip the ticket portion entirely — do not invent one and do not ask the user. Many projects don't use ticket IDs. Only ask if the user's request explicitly mentions a ticket system.

### 3. Determine commit scope — group or single

Analyze the changes. If they touch **different concerns** (e.g. a bug fix + a refactor, two unrelated features, docs + code), split into **multiple focused commits**. Each commit should be atomic and self-contained.

If all changes are related to one concern, create a single commit.

### 4. Determine commit type and scope

**Type** — pick one based on the nature of the change:

| Type       | When to use                                    |
|------------|------------------------------------------------|
| `Feat`     | New feature or capability                      |
| `Fix`      | Bug fix                                        |
| `Refactor` | Code restructuring without behavior change     |
| `Docs`     | Documentation only                             |
| `Style`    | Formatting, whitespace, missing semicolons     |
| `Test`     | Adding or updating tests                       |
| `CI`       | CI/CD configuration changes                    |
| `Chore`    | Build tooling, deps, housekeeping              |
| `Add`      | Adding new files, dependencies, assets         |
| `Change`   | Modifying existing behavior                    |
| `Remove`   | Removing code, files, or features              |

**Scope** — the area of the codebase affected, in parentheses. Pick a short noun derived from the actual path touched (e.g. `(api)`, `(ui)`, `(auth)`, `(db)`, `(docs)`). If the change spans several unrelated areas, split into multiple commits; if it spans closely related areas, pick the dominant one or omit the scope.

Scope is **optional** — if it adds no signal (e.g. single-folder repo, cross-cutting change), drop it: `Fix: Handle empty input`.

Use `!` after the scope only if there is a **breaking change**: `Feat(api)!:`

### 5. Write the commit message

#### Subject line format:

With ticket ID:

```
Type(scope): TICKET-ID Short description
```

Without ticket ID:

```
Type(scope): Short description
```

Without scope:

```
Type: Short description
```

Rules for the subject:

- **~50 characters** (5-8 words), max 72
- **Capitalize** the first word of the description
- **No period** at the end
- Answer the question: "What is this change good for?"
- Start the description with a verb matching the type (Add, Change, Remove, Fix, Refactor...)

Examples:
```
Feat(api): PROJ-123 Add product availability endpoint
Fix(ui): Handle empty cart state
Docs: Add deployment guide
Refactor(auth): ABC-456 Simplify token refresh flow
```

#### Body (optional but recommended for larger changes):

- Separate from subject with a **blank line**
- Wrap at **72 characters** per line (~10-12 words)
- Explain **what** and **why**, not how
- Use bullet points with `- ` prefix for lists
- Skip the body for trivial changes (typo fix, translation update)

#### Trailers (mandatory):

Every commit MUST end with a Co-Authored-By line. **Do not copy `{{MODEL}}` literally** — substitute your own running model identity.

Template:

```text
Co-Authored-By: {{MODEL}} <noreply@anthropic.com>
```

Rules for substituting `{{MODEL}}`:

- Write your actual model name and version as it is known at runtime. Format: `Claude <Family> <Version> (<context-window>)`, e.g. `Claude Opus 4.7 (1M context)`, `Claude Sonnet 4.6`, `Claude Haiku 4.5`.
- Do NOT hardcode a stale version. If you are Sonnet 4.6 running this skill, write Sonnet 4.6 — not whatever was in an example.
- If you genuinely do not know your own identity, write `Claude Code` (no version) — never invent a version.

#### Optional sections:

```
Related Issues: PROJ-123, PROJ-456
BREAKING CHANGE: Description of what breaks
```

### 6. Stage and commit

For each commit:

1. Stage only the relevant files: `git add <specific-files>`
2. Commit by piping the message over **stdin** with `git commit -F -`. This avoids all shell-quoting problems (double quotes, backticks, `$`, single quotes inside bullet points) that break a HEREDOC in `-m "$(cat <<EOF...)"`.

```bash
git commit -F - <<'MSG_END'
Type(scope): TICKET-ID Short description

- Bullet point explaining what changed and why.
- Another point if needed. "Quoted words" and $vars are safe here.

Co-Authored-By: {{MODEL}} <noreply@anthropic.com>
MSG_END
```

The outer heredoc delimiter must be quoted (`'MSG_END'`) so the shell does NOT expand `$vars` or backticks inside the message body. Remember to replace `{{MODEL}}` with your actual model identity (see trailer rules above).

### 7. Verify

Run `git log --oneline -3` and `git status` after committing to confirm success.

## Rules

- **Language:** Always write commit messages in **English**
- **Never push** to remote unless explicitly asked
- **Never stage secret files.** If `git status` / `git diff` lists a file matching a secret pattern (see step 1b), refuse to stage it; stop and ask the user.
- **Never read the content of a secret file** (the `disallowed-tools` list blocks the common patterns). Use path-based checks (`git check-ignore`, `git ls-files`) only. If a pattern is not in the block list but the file name suggests a secret, treat it as one anyway.
- **Never echo or include diff content from a secret file** in your output or commit message.
- **Always verify gitignore coverage** before committing (step 1b). Add missing patterns to `.gitignore` and commit that first (or bundle it) with user confirmation.
- **Never use `git add -A` or `git add .`** — always stage specific files
- **Never skip hooks** (no `--no-verify`)
- If a pre-commit hook fails, fix the issue and create a **new** commit (never `--amend`)

## Full commit example

```
Feat(api)!: PROJ-123 Add batch export endpoint

- Exposes POST /exports accepting up to 500 IDs per call.
- Asynchronous job; returns 202 + job ID, polled via GET /exports/{id}.
- Replaces the per-record GET /export which did not scale past
  a few thousand records.

Related Issues: PROJ-124, PROJ-125

BREAKING CHANGE: Removes GET /export; callers must migrate to
the async job API.

Co-Authored-By: {{MODEL}} <noreply@anthropic.com>
```