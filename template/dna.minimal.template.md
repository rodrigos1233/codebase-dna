# DNA (minimal)

<!--
  Minimal fingerprint for injection into CLAUDE.md or a subagent system prompt.
  Fill only axes where the project has a strong opinion.
  Remove axes with no clear position. Remove all comments before using.
  Target: fits within 200 lines when filled.
-->

## meta
project: ""
language: ""
stack: ""

## sliders
<!--
  Scale: 0–10. Include only axes with strong opinions.
  Add a one-line note when the position is non-obvious.
-->

coupling: #/10
abstraction-level: #/10
code-density: #/10
generality: #/10
encapsulation: #/10
configuration: #/10
service-granularity: #/10
data-consistency: #/10
synchrony: #/10
statefulness: #/10
test-coverage-posture: #/10 — aspirational: #/10
documentation-density: #/10
type-strictness: #/10
error-handling-loudness: #/10
defensive-programming: #/10
memory-speed-tradeoff: #/10
build-vs-buy-appetite: #/10
dependency-appetite: #/10

## binaries
pure-functions: true/false
idempotency: true/false
null-safety: true/false
determinism: true/false
encryption-at-rest: true/false
authentication-boundaries: true/false — mechanism: ""

## preferred-libs
http-server: ""
orm-or-query-builder: ""
validation: ""
authentication: ""
testing-unit: ""
testing-integration: ""
logging: ""
error-tracking: ""

## dead-ends
- ""

## simplicity-zones
- area: "" — rule: ""

## known-debt
- pattern: "" — location: "" — do not imitate because: ""
