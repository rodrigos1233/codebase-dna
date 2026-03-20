# Section Definitions

Each section below defines what it captures, the admission rules for entries, and one example.

The `evidence:` field is required on every entry in every section. Values:
- `observed` — seen directly in active code
- `documented` — stated in CLAUDE.md, README, decisions log, or inline comments
- `inferred` — reasoned from partial evidence; treat as a strong hint, not a hard rule

---

## stack

**What it captures:** The library or tool the project has committed to per problem domain. An agent scans this table to answer "which library do I use for X?" before writing any code.

**Admission rule:** Include a domain only if choosing the wrong library would create visible inconsistency in the codebase. Omit domains where the project has no strong opinion or where any reasonable choice would fit. This is not a dependency inventory.

**Format:** `domain: "library — brief note on how or where it is used"`

**Example:**
```yaml
stack:
  validation: {value: "zod — parse at route entry only, never in service layer", evidence: documented}
  testing-unit: {value: "vitest — all unit tests", evidence: observed}
```

Note: "jest is not used despite being in devDependencies" belongs in `known-debt`, not `stack`. Stack entries describe which library to use. A note about a present-but-wrong library choice is a debt entry.

---

## boundaries

**What it captures:** Structural constraints on what can call what, where authentication or validation is enforced, and placement rules. A placement rule is "new routes must be registered after the `authenticate()` call, not before" — it is a boundary constraint, not a coding idiom. Includes process boundaries — steps required outside the files being modified, such as updating a decisions log or a second file that must stay in sync.

**Admission rule:** Include an entry only if violating it produces a structural or security defect — not just a style inconsistency. Every entry must name the enforcement mechanism: a lint rule, a test, or an acknowledged convention.

**Format:** Rule + enforced-by + evidence.

**Example:**
```yaml
boundaries:
  - rule: "New HTTP routes must be registered after authenticate() in routes.ts, not before"
    enforced-by: "convention — no automated check; code review catches violations"
    evidence: documented
  - rule: "Domain types flow inward only: repositories may import domain types, domain types must not import repositories"
    enforced-by: "eslint-plugin-import boundary rules in .eslintrc"
    evidence: observed
```

---

## state-contracts

**What it captures:** Where state lives and which mechanism to use when. Change-coupling is a first-class concern: when you add X, you must also update Y and Z. This is some of the highest-value signal in the fingerprint — it describes invariants that span multiple files and are completely invisible from any one of them.

**Admission rule:** Include an entry only if getting it wrong causes a runtime defect, a sync failure, or a class of bugs that are hard to trace. If the state mechanism is obvious from the file being modified, omit it.

**Format:** State name + mechanism + rule + change-coupling (when applicable) + evidence.

**Example:**
```yaml
state-contracts:
  - state: "World simulation state"
    mechanism: "WorldState (in-memory) and SerializedWorldState (persistence) — both maintained in parallel"
    rule: "Never mutate WorldState directly; clone with structuredClone first, then modify the copy"
    change-coupling: "Any field added to WorldState requires a matching entry in SerializedWorldState and its serializer/deserializer"
    evidence: observed
```

---

## patterns

**What it captures:** Established idioms an agent would not infer from reading one file. These are deliberate conventions the team has committed to — not style preferences, but patterns whose violation produces real defects or inconsistency that is hard to notice locally.

**Admission rules — both gates must pass:**

1. **why-non-obvious:** Complete this field honestly. Why can't an agent infer this from reading the file being modified? If the pattern is visible in context, it does not belong here.
2. **Failure mode:** What wrong thing would an agent do if this entry were omitted? If there is no concrete wrong action, cut the entry. This gate is a discipline check for the auditor, not a recorded field.

**Aspirational fields** (`current:` / `target:`) are included only when there is an active migration and the distinction changes what new code should do. Omit if the pattern is stable.

**Format:** Name + rule + why-non-obvious + evidence + optional aspirational fields.

**Example:**
```yaml
patterns:
  - name: "Randomness injection"
    rule: "Inject randomFloat and similar non-deterministic values via config parameter, never call Math.random() inline"
    why-non-obvious: "Most call sites look like regular utility usage; the injection requirement is established by the test harness, not visible at the call site"
    evidence: observed
  - name: "i18n string addition"
    rule: "Add new strings to both en.json and es.json in the same commit; partial translations ship to production"
    why-non-obvious: "The i18n system silently falls back to the key name, so missing translations cause no runtime error"
    evidence: documented
    current: "Many legacy strings exist only in en.json"
    target: "All strings present in all locale files before merge"
```

---

## dead-ends

**What it captures:** Approaches that were tried, caused problems, and were intentionally abandoned. Recorded so future agents do not rediscover the same wrong path.

**Admission rule — relevance test first:** Would an agent plausibly reach for this pattern when working in this codebase? If not, it is project history, not a trap. A build tool replaced two years ago before the current team joined is not a dead end — it is trivia. Apply this test before checking for evidence.

**Evidence is then required.** At least one of:
- Explicit documentation, commit message, or comment stating the approach was abandoned
- A clear replacement pattern present in active code while the old approach is absent from new code
- Repeated evidence that new code avoids the pattern while old code still shows it

"This looks old" or "this seems outdated" is not sufficient. Auditors must not record folklore.

**Format:** Pattern + reason + replaced-by (if applicable) + evidence.

**Example:**
```yaml
dead-ends:
  - pattern: "Class-based React components with local state for data fetching"
    reason: "Tangled lifecycle logic and race conditions under concurrent updates; replaced during the React Query migration in Q3 2023"
    replaced-by: "React Query hooks in functional components"
    evidence: documented
```

---

## known-debt

**What it captures:** Patterns still present in the codebase that the team acknowledges are wrong and must not be imitated. These are not dead ends — the pattern has not been removed. They exist for historical reasons and are recorded so agents do not treat them as models.

**Admission rules:**
- Every entry must name a specific file, directory, or module in `location:`. "The repo sometimes does X badly" is not actionable.
- The entry must include what to do instead, not just what not to do.

**Aspirational fields** (`current:` / `target:`) are included when there is an active migration or a stated direction that changes what new code should do.

**Format:** Pattern + location (required) + why-it-exists + do-not-imitate + evidence + optional aspirational fields.

**Example:**
```yaml
known-debt:
  - pattern: "Raw SQL strings assembled with string concatenation"
    location: "legacy/reports/"
    why-it-exists: "Written before the query builder was adopted; not yet migrated because the reports module is rarely touched"
    do-not-imitate: "Use the query builder for all new queries, including queries added to this directory"
    evidence: observed
```

---

## simplicity-zones

**What it captures:** Areas intentionally kept simple that must not be abstracted, generalized, or otherwise "improved" in ways that add complexity.

**Admission rule:** This section should be rare. If there are many entries, the section loses meaning. It captures deliberate local anti-abstraction decisions — not general YAGNI sentiment and not the whole codebase's preference for simple code. Each entry must state a specific area and a specific rule, not a vague preference.

**Format:** Area + rule + evidence.

**Example:**
```yaml
simplicity-zones:
  - area: "Database migration scripts"
    rule: "Plain SQL only. No imports from application code. No shared helpers. Each migration must be fully self-contained and readable in isolation."
    evidence: documented
```
