---
name: contradiction-finder
description: >
  Step 2: Analyze all loaded papers and surface the most significant points
  where two or more authors make directly contradicting claims.
---

# Step 2: The Contradiction Finder

Analyze all loaded papers and surface the most significant points where two or more authors make directly contradicting claims.

## Rules

- Only flag **real contradictions** — mutually exclusive claims about the same phenomenon
- **Exclude** cases where differences are only in emphasis or scale
- Aim for **5-10 contradictions**. If fewer than 5 exist, list all and explain why there are few

## Output Format

Present results as a table:

| Contested Claim | Position A (Paper, Year) | Position B (Paper, Year) | Root Cause of Disagreement |
|-----------------|--------------------------|--------------------------|---------------------------|
| {claim} | {author, year}: {their position} | {author, year}: {their position} | {methodology / dataset / time period / definition / other} |

For root cause of disagreement, choose from: **methodology**, **dataset**, **time period**, **definition of terms**, or **other** (with explanation).

---

## Exit Criteria

- [ ] 5-10 contradictions identified (or all if fewer, with explanation)
- [ ] Each contradiction has both positions cited with paper references
- [ ] Root cause identified for each disagreement
- [ ] No false contradictions (emphasis/scale differences excluded)

---

## Anti-Rationalization Table

| Excuse | Reality |
|--------|---------|
| "The papers mostly agree" | Disagreements exist at methodology, scope, or definition level. Dig deeper. |
| "This is a nuance, not a contradiction" | If both claims cannot be simultaneously true, it's a contradiction. |
| "I need to summarize the papers first" | You already have the processing protocol. Work from the loaded papers directly. |
