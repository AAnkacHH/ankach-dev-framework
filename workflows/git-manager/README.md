# Git Manager

> Git workflow specialist — routes branching, commits, PRs, sync, cleanup, history search, stash, and revert through dedicated safety-hardened skills.

A set of Claude Code skills and one orchestrating agent (`git-agent`) that encapsulates the daily git workflow: creating branches, committing, syncing with `main`/`master`/`dev`/`stage`, opening PRs, cleaning up, searching history, working with stashes, and reverting changes.

Every convention (branch name format, commit message layout, PR template, sensitive-file list) **lives in the matching skill as the single source of truth** — the agent only routes and orchestrates.

## Structure

```text
workflows/git-manager/
├── README.md                        (this file)
├── agents/
│   └── git-agent.md                 orchestrator agent (model: sonnet)
├── commands/ankach/
│   ├── commit.md
│   ├── create-branch.md
│   ├── sync-branch.md
│   ├── create-pr.md
│   ├── cleanup-branches.md
│   ├── search-commits.md
│   ├── stash-manager.md
│   └── revert.md
└── skills/
    ├── _shared/
    │   └── secret-patterns.md       SoT for sensitive-file patterns
    ├── commit/
    ├── create-branch/
    ├── create-pr/
    ├── sync-branch/
    ├── cleanup-branches/
    ├── search-commits/
    ├── stash-manager/
    └── revert/
```

## Installation

**Claude Code looks for agents and skills under `.claude/` in the target project.** This repository's `workflows/git-manager/` is not usable on its own — you need to **copy (or symlink) the contents into `.claude/` of the target project**, otherwise `git-agent` and the individual skills are not discoverable.

### Target layout

After installation, the target project must have:

```text
<project>/
└── .claude/
    ├── agents/
    │   └── git-agent.md
    ├── commands/ankach/
    │   ├── commit.md
    │   ├── create-branch.md
    │   ├── sync-branch.md
    │   ├── create-pr.md
    │   ├── cleanup-branches.md
    │   ├── search-commits.md
    │   ├── stash-manager.md
    │   └── revert.md
    └── skills/
        ├── _shared/
        │   └── secret-patterns.md
        ├── commit/SKILL.md
        ├── create-branch/SKILL.md
        ├── create-pr/SKILL.md
        ├── sync-branch/SKILL.md
        ├── cleanup-branches/SKILL.md
        ├── search-commits/SKILL.md
        ├── stash-manager/SKILL.md
        └── revert/SKILL.md
```

### Option A — copy (recommended for production projects)

Reliable, self-contained — the target project gets its own copy and doesn't depend on this source repo.

```bash
# from the target project root
mkdir -p .claude/agents .claude/skills .claude/commands/ankach

cp <path>/workflows/git-manager/agents/git-agent.md .claude/agents/
cp -r <path>/workflows/git-manager/skills/* .claude/skills/
cp <path>/workflows/git-manager/commands/ankach/*.md .claude/commands/ankach/
```

Commit `.claude/` into the target project's git history so the workflow is available to all team members.

### Option B — symlink (for development / testing)

Symlinks keep the target project tied to the live source — when you edit something in `workflows/git-manager/`, every linked project sees it immediately. Downside: if you move or delete this directory, every linked project breaks.

```bash
# from the target project root
mkdir -p .claude
ln -s <abs-path>/workflows/git-manager/agents .claude/agents
ln -s <abs-path>/workflows/git-manager/skills .claude/skills
ln -s <abs-path>/workflows/git-manager/commands/ankach .claude/commands/ankach
```

Don't commit symlinks — each developer creates them locally.

### Verifying the install

1. Restart the Claude Code session in the target project (agents/skills load at startup).
2. In the parent window run `/agents` — `git-agent` should be listed.
3. Type `/` and start typing `/commit` — the slash command should appear with a description.
4. Try: "ask git-agent to show me the repo status" — the agent should respond with a single short line.

If any of the above fails, check:

- `.claude/agents/git-agent.md` and `.claude/skills/<name>/SKILL.md` exist at the exact path.
- YAML frontmatter is intact (matching `---` delimiters).
- You restarted the session after installing.

## How to use it

### Mode 1 — via the agent (strongly recommended)

`git-agent` is designed to be called from the main Claude Code window (typically Opus) using the Agent tool. The agent picks the right skill, runs it, and returns the minimum number of tokens to the parent context.

**Why call the agent instead of skills directly:**

1. **Token savings in the main window.** When you call a skill directly from the Opus window, every bash output (`git status`, `git diff`, `git log`, stderr), every `AskUserQuestion` prompt, and every intermediate result gets injected into the Opus context. A `sync-branch` with a conflict can easily be 5–15k tokens. The agent absorbs all that and returns only 1–3 lines on success (PR URL, new SHA, branch name) or a structured error report on failure. A single agent call typically saves 80–95% of tokens vs. a direct skill invocation from Opus.

2. **Cheaper model.** The agent runs on Sonnet; Opus only orchestrates. Sonnet handles git decisions (routing, ticket-ID validation, base-branch detection, clarifying questions) without trouble. Delegating git work to the agent reduces API cost proportionally to how much git work you do in a session.

3. **Cache stays warm.** When Opus only sends a short query to the agent and gets a short answer back, the prompt cache in the main window stays intact. Direct skill calls from Opus heat up and fragment the cache across many tool results.

4. **Consistent safety.** The agent enforces safety rules (no force-push to prod, no destructive ops without confirmation) even if the user forgets. Direct skill calls skip this outer layer.

**When to call a skill directly:**

- Debugging or extending a specific skill.
- When you know exactly what you want and don't want the agent to ask questions (e.g. `/commit` in an automated hook).
- In a test / dev environment where token cost doesn't matter.

Example prompts the agent understands:

- "start a new task: add product availability display"
- "commit these changes"
- "open a PR"
- "rebase this branch onto main"
- "who deleted `calculateBundlePrice`?"
- "clean up merged branches"
- "revert the last PR"

At every decision point (rebase vs merge, base branch, destructive op) the agent asks via `AskUserQuestion` with concrete options — never open-ended.

### Mode 2 — direct skill invocation

Each skill is also callable as a standalone `/slash` command (e.g. `/ankach:commit`, `/ankach:create-branch`). Useful when you know exactly what you want and don't want to go through the agent.

## Skill list

| Skill | What it does | Input conventions |
|---|---|---|
| `commit` | Creates one or more focused atomic commits using conventional-commit style. Checks that no sensitive files appear in the diff, and can append missing patterns to `.gitignore`. | `Type(scope): [TICKET-ID] Short description` + `Co-Authored-By: {{MODEL}}` trailer |
| `create-branch` | Creates a new branch off the correct long-lived base. Asks for type (`feature`/`bugfix`/`hotfix`) and extracts any ticket ID from the input if present. | `<type>/<ticket-or-slug>-<short-desc>` (English, kebab-case) |
| `sync-branch` | Updates the current branch against the correct long-lived base. `hotfix/*` always against `main`/`master`; `feature/*` and `bugfix/*` ask between `dev`/`stage` and `main`/`master`. Always asks rebase vs merge. | Rule: if the tree was stashed, explicitly remind about `git stash pop` AFTER `rebase --continue`. |
| `create-pr` | Opens a PR via the `gh` CLI. Generates a title and a body with Summary + Test plan. Extracts ticket ID from the branch if present. | Title format: `[TICKET-ID] Short description`, body with `## Summary` and `## Test plan`. |
| `cleanup-branches` | Finds local branches merged into the default branch and offers to delete them. Always dry-run → confirm → `-d` (never `-D`). Never deletes remote branches. | Ignores `main`, `master`, `dev`, `stage`, `release/*`, `hotfix/*`, and the current branch. |
| `search-commits` | Universal read-only history search. Includes a toolbox (pickaxe `-S`, regex `-G`, `--grep`, `--author`, path history, blame, `git log -L`, GitHub search). | Prefers `git log --all --grep='<term>'`, pickaxe for vanished code. |
| `stash-manager` | Stash operations: save, list, show, pop, apply, drop, clear. `drop` requires an explicit `stash@{N}`; `clear` requires a confirmed listing. | Recommends `apply` over `pop` when unsure. |
| `revert` | Creates a revert commit (additive, no history rewrite). Single commit, range, or merge commit (`-m 1`). | Refuses to run on a long-lived branch — requires a working branch. |

## Branch and commit conventions (brief)

The full rule lives in the relevant skill. Orientation:

- **Long-lived branches:** `main` (modern default) / `master` (legacy repos) = prod; `dev` / `stage` = staging. Detected from `origin`, never hard-coded.
- **Working branches:** `feature/`, `bugfix/`, `hotfix/`. `hotfix` always branches off `main`/`master`; the others follow the project flow.
- **Ticket ID:** optional. If the user or branch name contains a recognizable pattern (`[A-Z]+-\d+` for Jira/Linear, `#\d+` for GitHub Issues), it gets used; otherwise commits/PRs omit it.
- **Language:** branch names, commit messages and PR text in **English only**.
- **Rebase vs merge:** always asked, never silently picked.

## Safety

Safety is layered; each layer catches a different class of risk.

1. **Narrow allowlist is the primary defense.** Each skill lists only the specific `Bash(git ...)` / `Bash(gh ...)` patterns it actually uses. Anything outside that list — `python -c`, `curl ...`, `cat ...`, `git show <sha>`, `git blame <file>`, `git log -p` — falls through to a user prompt rather than running automatically. No approval → no execution.
2. **Hard-deny reserved for allow-adjacent risks.** `disallowed-tools` only blocks commands that the allowlist would otherwise let through. Examples:
   - `commit` allows `Bash(git add *)` → hard-denies `Bash(git add *<secret>*)` for every pattern in `_shared/secret-patterns.md`.
   - `cleanup-branches` allows `Bash(git branch -d *)` → hard-denies `git branch -d main/master/dev/stage/release/*/hotfix/*`.
   - `create-pr` allows `Bash(git push origin *)` → hard-denies every `--force` variant and `gh pr create --body-file` (content-leak via PR body).
   - `sync-branch` allows `Bash(git rebase origin/*)` → hard-denies `git rebase * --exec*`.
   - `search-commits` has a narrow allowlist for `git log`/`show`/`blame` (e.g. `--stat`, `--name-only`, `--oneline`, `-L`) — bare `git show <sha>:<path>`, `git blame <file>`, `git log -p` are not in the list → prompts.
3. **Defense-in-depth on dangerous ops.** Commands that should never run regardless of user approval are hard-denied in every skill: `git filter-branch`, `git filter-repo`, `git replace`, `git update-ref`, `git fsck`, `git gc`, `git config --global/--system`, `git remote set-url`.
4. **Agent-level catchall.** `git-agent` uses blanket `Bash` (to orchestrate any skill), so its `disallowed-tools` is the full defensive block: shell escapes (`bash`, `python`, `curl`, `cat`, …), all `Read(**/<secret>)` patterns, content-leaking git forms. The agent is the only file where the full block lives.
5. **Sensitive-file list** in `skills/_shared/secret-patterns.md` is the single source of truth for what counts as a secret. The `commit` skill and the agent replicate relevant entries into their `disallowed-tools`; other skills rely on the prompt-fallback since they don't list `Read` or `Grep` in allow.
6. **Prompt injection** — branch names, commit messages, PR bodies, SHA arguments, and stash messages are treated as untrusted input. `revert` has explicit SHA validation prose; `stash-manager` warns about message quoting; `create-pr` mandates a quoted heredoc for PR body.

## Output format

The agent returns the parent context the minimum of tokens:

- **On success** — 1–3 lines, artifacts only (PR URL, new SHA, branch name).
- **On failure** — full context: what was attempted, exact error, repo state, 2–3 concrete recovery options.

That way the main (Opus) context doesn't pick up thousands of tokens per git call.

## Extending

### Add a new sensitive-file pattern

1. Add one line to `skills/_shared/secret-patterns.md` with a reason.
2. Add the pattern to `agents/git-agent.md` `disallowed-tools`:

   ```yaml
     - Read(**/<pattern>)
   ```

3. Add the pattern to `commit` skill's `disallowed-tools` (both `git add` and `Read` — `commit` is the only skill with blanket `Read` allow):

   ```yaml
     - Bash(git add *<pattern>*)
     - Read(**/<pattern>)
   ```

4. Other skills **don't** need the entry — their allowlists don't include `Read`/`Grep`, so reads of the new pattern fall through to a user prompt.

### Add a new skill

1. Create `skills/<skill-name>/SKILL.md` with frontmatter: `name`, `description`, narrow `allowed-tools` (list only the specific `Bash(git <subcommand> ...)` patterns the skill needs — no blanket `Bash`), `disallowed-tools`.
2. Keep `disallowed-tools` lean: list only hard-denies where the allowlist is broader than safe (e.g. if you allow `Bash(git push *)`, hard-deny force-push variants). Everything else falls through to prompt-on-use.
3. Always hard-deny the dangerous-ops block: `git filter-branch`, `git filter-repo`, `git replace`, `git update-ref`, `git fsck`, `git gc`, `git config`, `git remote set-url`.
4. Keep the same report style: success = 1–3 lines, failure = full context.
5. Add a row to the skill table in `agents/git-agent.md` routing section.
6. Add a thin pointer `commands/ankach/<skill-name>.md` so the skill is invocable via `/ankach:<skill-name>`.

### Update a convention (e.g. branch format)

Conventions belong to the owning skill — update there, never in the agent:

- Branches → `create-branch/SKILL.md`
- Commit message → `commit/SKILL.md`
- PR template → `create-pr/SKILL.md`
- Rebase / merge policy → `sync-branch/SKILL.md`

## Workflow examples

### From zero to PR

```text
1. "start a new task: add product availability"
   → agent: invoke create-branch (asks: feature/bugfix/hotfix? → feature)
   → result: branch feature/add-product-availability (from main)

2. ...you make changes...

3. "commit"
   → agent: invoke commit (detects sensitive files, offers to update .gitignore, builds the commit)

4. "open a PR"
   → agent:
      - sync-branch (asks: dev / main?)
      - create-pr (push + PR body + URL)
```

### Cleanup after a successful merge

```text
"clean up merged branches"
  → cleanup-branches: dry-run → list → confirm → delete 5 local branches via `-d`
```

### Revert a deployed change

```text
"revert the last availability PR"
  → revert: finds the merge SHA, creates hotfix/revert-availability, asks for the mainline parent, makes the revert commit
  → next: create-pr
```

## Out of scope

Intentionally outside this workflow (use plain git or other tools):

- **cherry-pick** between branches
- **bisect** to find the commit that broke the build
- **interactive rebase** (reordering / squashing your own commits)
- **reflog recovery** after a destructive operation
- **git worktree** management
- **Deleting remote branches** — do this manually via GitHub/Bitbucket UI
- **History-rewriting operations** (`filter-branch`, `filter-repo`) — if a secret ended up in history, the skills will point you at a manual scrub plus credential rotation
