---
name: documentation
description: >
  Phase 8: Generate API docs, dev notes, and changelog entry from implemented code.
  Source of truth is the code, not design docs. Use after Phase 7 Review passes (SHIP).
  MUST complete before Phase 9: Deploy.
---

# Phase 8: Documentation

<HARD-GATE>
Requires Phase 7 Review with verdict SHIP.
Read `.planning/active/07-review/index.md` — if verdict is not SHIP, stop.
</HARD-GATE>

---

## Step 1: Context (Silent)

Read:
1. `.planning/active/07-review/index.md` — confirm SHIP
2. `.planning/active/03-design/api-contracts.md` — designed endpoints
3. Actual implemented code — `git diff main...HEAD --name-only`
4. `.planning/STATE.md`

**Key principle: Generate documentation from CODE (source of truth), not from design docs.** Design docs may have drifted during implementation.

---

## Step 2: API Docs

For each new or changed endpoint, document:

```markdown
### {METHOD} {path}

**Purpose:** {what it does}
**Auth:** {required/public}

**Request:**
| Parameter | Location | Type | Required | Description |
|-----------|----------|------|----------|-------------|

**Response (200):**
```json
{example response}
```

**Errors:**
| Status | Code | When |
|--------|------|------|

**Example:**
```bash
curl -X {METHOD} {path} -H "Authorization: Bearer ..." -d '{...}'
```
```

Write to `.planning/active/08-documentation/api-docs.md`.

---

## Step 3: Dev Notes

Document how the implementation works for future developers/agents:

```markdown
# Dev Notes: {Feature Name}

## How It Works
{1-2 paragraphs: data flow, key components, non-obvious decisions}

## Key Files
| File | Purpose |
|------|---------|
| `src/.../` | {what it does} |

## Non-Obvious Decisions
- {decision}: {why it was made this way, not the obvious way}

## Gotchas
- {thing that might surprise a future developer}
```

Write to `.planning/active/08-documentation/dev-notes.md`.

---

## Step 4: Changelog Entry

Write a changelog entry following Keep a Changelog format:

```markdown
## [{version}] - {YYYY-MM-DD}

### Added
- {new feature}

### Changed
- {modified behavior}

### Fixed
- {bug fix}
```

Write to `.planning/active/08-documentation/changelog-entry.md`.

---

## Step 5: Approval + Document

### Write `.planning/active/08-documentation/index.md`:

```markdown
# Phase 8: Documentation — {Feature Name}

| Field    | Value                    |
|----------|--------------------------|
| Phase    | 8 of 9                  |
| Status   | APPROVED                |
| Date     | {YYYY-MM-DD}            |

## Summary
{What was documented}

## Artifacts
- API docs: {N} endpoints documented
- Dev notes: key components and gotchas
- Changelog: entry for {version}

## Files in This Phase
- [api-docs.md](./api-docs.md)
- [dev-notes.md](./dev-notes.md)
- [changelog-entry.md](./changelog-entry.md)

## Gate
Approved → Phase 9: Deploy
```

Present to user:

```
Documentation saved to `.planning/active/08-documentation/`.
- API docs: {N} endpoints documented
- Dev notes: key files and gotchas
- Changelog entry ready

`approve` → Phase 9: Deploy | `reject` → tell me what to fix.
```

<HARD-GATE>
Wait for approval. On approve → update STATE.md.
</HARD-GATE>

---

## Exit Criteria (ALL must be true)

- [ ] API docs written for all new/changed endpoints (Step 2)
- [ ] Dev notes written with key files and gotchas (Step 3)
- [ ] Changelog entry written (Step 4)
- [ ] All artifacts written to `.planning/active/08-documentation/`
- [ ] User approved
- [ ] STATE.md updated

---

## Anti-Rationalization Table

| Excuse | Reality |
|--------|---------|
| "The code is self-documenting" | Code explains HOW, docs explain WHY and for WHOM. |
| "Nobody reads docs anyway" | Future agents read docs. Future you reads docs. |
| "Changelog is unnecessary for small changes" | Small changes add up. Track them all. |
