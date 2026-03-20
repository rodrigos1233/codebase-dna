# DNA Fingerprint

<!--
  This fingerprint describes how the codebase is written, not what it does.
  It captures information an agent cannot see by reading a single file.
  If a section has nothing to record, write [] for list sections or omit the section.
  Remove instruction comments before committing.
-->

## meta

```yaml
project: ""              # repo or project name
audited-by: ""           # "human", "agent", or "human + agent"
audited-at: ""           # ISO date: YYYY-MM-DD
codebase-version: ""     # git commit SHA or version tag at time of audit
```

## audit-notes

<!--
  Contradictions found, areas with insufficient evidence, or areas not sampled.
  Leave blank if none.
-->

---

## stack

<!--
  Libraries and tools the project has committed to per domain.
  Admission: include a domain only if choosing the wrong library would create
  visible inconsistency. This is not a dependency inventory.
  Format: domain: "library — brief note on how or where it is used"
  evidence field applies to the whole stack section, not per entry.
-->

```yaml
stack:
  # Core
  http-server: {value: "", evidence: ""}    # observed | documented | inferred
  routing: {value: "", evidence: ""}
  validation: {value: "", evidence: ""}
  orm-or-query-builder: {value: "", evidence: ""}
  database-migrations: {value: "", evidence: ""}

  # Auth
  authentication: {value: "", evidence: ""}
  authorization: {value: "", evidence: ""}

  # Testing
  testing-unit: {value: "", evidence: ""}
  testing-integration: {value: "", evidence: ""}
  testing-e2e: {value: "", evidence: ""}

  # Observability
  logging: {value: "", evidence: ""}
  error-tracking: {value: "", evidence: ""}

  # Add or remove domains as needed for this project
```

---

## boundaries

<!--
  Structural constraints on what can call what, where auth/validation is enforced,
  and placement rules (e.g., "new routes registered after authenticate(), not before").
  Include the enforcement mechanism: lint rule, test, or convention.
  evidence: observed | documented | inferred
-->

```yaml
boundaries:
  - rule: ""
    enforced-by: ""   # "eslint-plugin-import", "test in boundaries.test.ts", "convention"
    evidence: ""      # observed | documented | inferred
```

---

## state-contracts

<!--
  Where state lives and which mechanism to use when.
  Change-coupling is a first-class concern: "when you add X, you must also update Y and Z."
  Include an entry only if violating it causes a runtime defect or a hard-to-trace bug.
  evidence: observed | documented | inferred
-->

```yaml
state-contracts:
  - state: ""
    mechanism: ""        # e.g., "Redux store via useSelector/useDispatch"
    rule: ""             # what an agent must do when touching this state
    change-coupling: ""  # omit if no coupling requirement exists
    evidence: ""         # observed | documented | inferred
```

---

## patterns

<!--
  Established idioms an agent would not infer from reading one file.
  TWO admission gates — both must pass:
    1. why-non-obvious: complete this honestly. If you cannot, cut the entry.
    2. Failure mode check (not a field): what wrong thing would an agent do without this entry?
       If there is no concrete wrong action, cut the entry.
  Aspirational fields (current/target) only when an active migration changes what new code should do.
  evidence: observed | documented | inferred
-->

```yaml
patterns:
  - name: ""
    rule: ""
    why-non-obvious: ""
    evidence: ""         # observed | documented | inferred
    # current: ""        # omit unless there is an active migration
    # target: ""
```

---

## dead-ends

<!--
  Patterns tried and abandoned. Evidence required — at least one of:
    - Explicit documentation, comment, or commit message stating abandonment
    - A clear replacement present in active code while old approach is absent from new code
    - Repeated evidence new code avoids the pattern while old code still shows it
  "This looks old" is not sufficient. Do not record folklore.
  evidence: observed | documented | inferred
-->

```yaml
dead-ends:
  - pattern: ""
    reason: ""
    replaced-by: ""  # omit if nothing replaced it
    evidence: ""     # observed | documented | inferred
```

---

## known-debt

<!--
  Patterns still present in the codebase that must not be imitated.
  Every entry must name a specific file, directory, or module in location:.
  "The repo sometimes does X badly" is not actionable.
  Aspirational fields (current/target) when there is an active direction.
  evidence: observed | documented | inferred
-->

```yaml
known-debt:
  - pattern: ""
    location: ""       # file, directory, or module — required
    why-it-exists: ""
    do-not-imitate: "" # what to do instead, not just what not to do
    evidence: ""       # observed | documented | inferred
    # current: ""      # omit unless there is an active migration
    # target: ""
```

---

## simplicity-zones

<!--
  Areas intentionally kept simple that must not be abstracted or improved.
  This section should be rare. Many entries means the section has lost meaning.
  Captures deliberate local anti-abstraction rules — not general YAGNI sentiment.
  evidence: observed | documented | inferred
-->

```yaml
simplicity-zones:
  - area: ""
    rule: ""
    evidence: ""  # observed | documented | inferred
```
