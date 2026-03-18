# DNA (minimal)

Reusable, persistent compact fingerprint for injection into a system prompt or `CLAUDE.md`.
Use this for long-lived includes; use `dna.inject.md` for task-scoped, freshly generated injection.
Keep only rules that change implementation decisions.

## meta
project: ""
language: ""
stack: ""

## preferred libs/domains
preferred-libs: ""  # only durable choices
preferred-domains: ""  # only durable choices

## critical rules
### do not use
- ""

### keep simple
- area: "" — rule: ""

### known debt
- pattern: "" — location: "" — do not imitate because: ""

## code-shaping constraints
pure-functions: true/false — scope: ""
idempotency: true/false — scope: ""
null-safety: true/false — scope: ""
determinism: true/false — scope: ""
encryption-at-rest: true/false — scope: ""
authentication-boundaries: true/false — scope: "" — mechanism: ""
type-strictness: ""  # e.g. "strict - no any"
error-handling-loudness: ""  # e.g. "fail fast with explicit logs"
