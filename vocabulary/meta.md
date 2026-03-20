# Meta Concepts

These concepts apply across all sections. They are qualifiers that modify how entries should be read and written — not additional sections.

---

## Evidence Levels

Every entry in every section carries an `evidence:` field. This field is what distinguishes a trustworthy fingerprint from a confident-sounding guess.

| Value | Meaning |
|---|---|
| `observed` | Seen directly in active code — the pattern is present and in use |
| `documented` | Stated in CLAUDE.md, README, a decisions log, commit message, or inline comment |
| `inferred` | Reasoned from partial evidence — a strong hint based on what was read, not confirmed |

**How agents should use evidence levels:**

- `observed` and `documented` entries are hard rules. Follow them.
- `inferred` entries are strong hints. Follow them unless you observe clear contradictory evidence in the files you are working with. If you contradict an `inferred` entry, note it.
- When multiple entries in the same section conflict, prefer `documented` over `observed` (documented intent over current implementation) and both over `inferred`.

---

## Aspirational Markers

Aspirational markers appear only in `patterns` and `known-debt` entries, and only when there is an active migration or stated direction that changes what new code should do. If the distinction does not change agent behavior, omit the markers.

**Fields:**
- `current:` — describes what the codebase does now
- `target:` — describes where it is moving
- `do-not-imitate:` — explicit instruction for new code (used in `known-debt`)

**How to write:**

```yaml
# In known-debt:
- pattern: "Error handling via thrown exceptions in service layer"
  location: "src/services/"
  why-it-exists: "Predates the Result type adoption; gradual migration underway"
  do-not-imitate: "Use Result<T, E> returns for all new service functions"
  evidence: documented
  current: "Most services throw; callers use try/catch"
  target: "Result<T, E> typed returns throughout"

# In patterns:
- name: "State mutation"
  rule: "Use structuredClone before mutating world state"
  why-non-obvious: "Direct mutation causes aliasing bugs that don't surface in unit tests"
  evidence: observed
  current: "Older modules mutate directly — do not imitate"
  target: "All state mutation goes through clone-then-modify"
```

**How agents should use aspirational markers:**

- Write new code toward `target:`, not to match `current:`.
- Use `current:` to calibrate what you will find in existing code — do not be surprised by the old pattern, and do not propagate it.

---

## Drift

Drift occurs when code you observe contradicts an entry in the fingerprint. It is not a bug — it is information. Drift means either the fingerprint is outdated, or the code is moving away from the team's stated conventions.

**When to flag drift:**

- You observe a pattern that directly contradicts a `boundaries`, `state-contracts`, or `patterns` entry
- A `dead-ends` entry describes a pattern you find in active use in new-looking code
- An `inferred` entry turns out to be wrong based on what you read

**How to report drift:**

Do not silently pick one signal over the other. Surface it:

```
Drift detected: dna.md records that randomFloat is injected via config (patterns: "Randomness injection"),
but two recently modified files in src/simulation/ call Math.random() directly.
This may mean the pattern is aspirational, or that these files predate the convention.

Proceeding with injection as stated in the fingerprint. Flagging for review.
```

**When to update the fingerprint:**

If drift is widespread and represents a real change in direction, update the relevant entry — change `evidence: inferred` to `observed` if the evidence has shifted, or add aspirational markers if the team is intentionally moving away from the current state.
