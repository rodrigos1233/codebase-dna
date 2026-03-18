# DNA Fingerprint

<!--
  Instructions for filling this template:
  - position: (sliders) a number 0–10. (binaries) true or false.
  - descriptive: what the codebase currently does. Required when position is filled.
  - aspirational: what it should do. Omit if same as descriptive.
  - notes: justification, exceptions, or agent guidance. Required for extreme positions (0–2 or 8–10) and inconsistency.
  - Remove the instruction comments before committing this file.
-->

## meta

```yaml
project: ""                # Project or repo name
audited-by: ""             # "human", "agent", or "human + agent"
audited-at: ""             # ISO date: YYYY-MM-DD
codebase-version: ""       # git commit SHA or version tag at time of audit
```

## audit-notes

<!--
  Contradictions found, areas with insufficient evidence, axes where the
  code and documentation disagreed. Leave blank if none.
-->

---

## sliders

### coupling
```yaml
coupling:
  position:           # 0 = tightly coupled, 10 = loosely coupled
  descriptive: >
  aspirational: >
  notes: >
```

### abstraction-level
```yaml
abstraction-level:
  position:           # 0 = close to the machine, 10 = domain language
  descriptive: >
  aspirational: >
  notes: >
```

### code-density
```yaml
code-density:
  position:           # 0 = sparse, 10 = dense
  descriptive: >
  aspirational: >
  notes: >
```

### generality
```yaml
generality:
  position:           # 0 = specific / YAGNI, 10 = general / framework-oriented
  descriptive: >
  aspirational: >
  notes: >
```

### encapsulation
```yaml
encapsulation:
  position:           # 0 = open / reach-through, 10 = strict public interface
  descriptive: >
  aspirational: >
  notes: >
```

### configuration
```yaml
configuration:
  position:           # 0 = hardcoded, 10 = fully configurable
  descriptive: >
  aspirational: >
  notes: >
```

### service-granularity
```yaml
service-granularity:
  position:           # 0 = monolith, 10 = microservices
  descriptive: >
  aspirational: >
  notes: >
```

### data-consistency
```yaml
data-consistency:
  position:           # 0 = eventual consistency, 10 = strong consistency
  descriptive: >
  aspirational: >
  notes: >
```

### synchrony
```yaml
synchrony:
  position:           # 0 = synchronous, 10 = asynchronous
  descriptive: >
  aspirational: >
  notes: >
```

### statefulness
```yaml
statefulness:
  position:           # 0 = stateless, 10 = stateful
  descriptive: >
  aspirational: >
  notes: >
```

### test-coverage-posture
```yaml
test-coverage-posture:
  position:           # 0 = minimal testing, 10 = comprehensive
  descriptive: >
  aspirational: >
  notes: >
```

### documentation-density
```yaml
documentation-density:
  position:           # 0 = minimal docs, 10 = heavily documented
  descriptive: >
  aspirational: >
  notes: >
```

### type-strictness
```yaml
type-strictness:
  position:           # 0 = untyped / any, 10 = strictly typed / invariants enforced
  descriptive: >
  aspirational: >
  notes: >
```

### error-handling-loudness
```yaml
error-handling-loudness:
  position:           # 0 = silent / swallowed, 10 = loud / fail-fast
  descriptive: >
  aspirational: >
  notes: >
```

### defensive-programming
```yaml
defensive-programming:
  position:           # 0 = trusting, 10 = defensive
  descriptive: >
  aspirational: >
  notes: >
```

### memory-speed-tradeoff
```yaml
memory-speed-tradeoff:
  position:           # 0 = compute on demand, 10 = cache aggressively
  descriptive: >
  aspirational: >
  notes: >
```

### build-vs-buy-appetite
```yaml
build-vs-buy-appetite:
  position:           # 0 = build everything, 10 = buy / adopt everything
  descriptive: >
  aspirational: >
  notes: >
```

### dependency-appetite
```yaml
dependency-appetite:
  position:           # 0 = dependency-averse, 10 = dependency-friendly
  descriptive: >
  aspirational: >
  notes: >
```

### data-flow-patterns
```yaml
data-flow-patterns:
  position:           # 0 = ad hoc flow, 10 = disciplined flow
  descriptive: >
  aspirational: >
  notes: >            # cover: where transformation happens, data shape at boundaries, result type pattern
```

---

## binaries

### pure-functions
```yaml
pure-functions:
  value:              # true or false
  scope: ""           # e.g., "utility layer only" — omit if applies globally
  notes: >
```

### idempotency
```yaml
idempotency:
  value:              # true or false
  scope: ""           # e.g., "write endpoints and background jobs"
  notes: >
```

### null-safety
```yaml
null-safety:
  value:              # true or false
  scope: ""           # e.g., "server-side TypeScript only" — omit if applies globally
  notes: >
```

### determinism
```yaml
determinism:
  value:              # true or false
  scope: ""           # e.g., "domain and service layers — infrastructure layer excluded"
  notes: >
```

### encryption-at-rest
```yaml
encryption-at-rest:
  value:              # true or false
  scope: ""           # e.g., "PII fields only" or "full disk via cloud provider"
  notes: >
```

### authentication-boundaries
```yaml
authentication-boundaries:
  value:              # true or false
  scope: ""           # e.g., "HTTP API — internal gRPC calls use mTLS separately"
  notes: >
  mechanism: ""       # e.g., "JWT middleware applied globally in routes.ts"
```

---

## preferred-libs

<!--
  Record the library or approach used per problem domain.
  Include only domains where the project has a real opinion.
  Remove rows that don't apply. Add rows the project needs.
  Format: domain: "library-name — brief note on how it is used"
-->

```yaml
preferred-libs:
  # Core
  http-server: ""
  routing: ""
  orm-or-query-builder: ""
  database-migrations: ""
  validation: ""

  # Auth
  authentication: ""
  authorization: ""

  # Testing
  testing-unit: ""
  testing-integration: ""
  testing-e2e: ""
  mocking: ""

  # Observability
  logging: ""
  error-tracking: ""
  metrics: ""

  # Infrastructure (include only if the project has opinions)
  caching: ""
  job-queue: ""
  env-config: ""
```

---

## dead-ends

<!--
  Patterns tried and abandoned. Format each entry as shown.
  Leave the list empty ([]) if none.
-->

```yaml
dead-ends:
  - pattern: ""
    reason: >
    replaced-by: ""
```

---

## simplicity-zones

<!--
  Areas intentionally kept simple that must not be abstracted or generalized.
  Leave the list empty ([]) if none.
-->

```yaml
simplicity-zones:
  - area: ""
    rule: >
```

---

## known-debt

<!--
  Patterns that exist in the codebase but should not be imitated.
  These are not dead ends (which were abandoned) — they are still present
  but represent acknowledged violations of the team's own standards.
  Leave the list empty ([]) if none.
-->

```yaml
known-debt:
  - pattern: ""
    location: ""       # file, directory, or module where it appears
    why-it-exists: >
    do-not-imitate: >
```
