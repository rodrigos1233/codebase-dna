# Sliders

Each axis is a spectrum from 0 (left extreme) to 10 (right extreme). Positions are not good or bad — they describe tradeoffs. A position of 5 means the codebase sits in the middle, not that it has no opinion.

---

## coupling

**Definition:** How much modules depend on each other's internals vs communicating through stable interfaces.

| 0 — Tightly Coupled | 10 — Loosely Coupled |
|---|---|
| Modules import and call each other directly. Changing one module requires changing others. Circular imports are common. | Modules interact through interfaces, events, or message passing. Each module can be swapped or tested independently. |

**Tradeoff:** Tight coupling is faster to write and easier to trace. Loose coupling is easier to test, replace, and scale independently.

**How to identify:** Count direct imports between non-trivial modules. Look for circular dependencies, shared mutable state accessed across package boundaries, or tests that require instantiating large swaths of the system.

---

## abstraction-level

**Definition:** How close the code is to the domain problem vs the machine.

| 0 — Low Abstraction | 10 — High Abstraction |
|---|---|
| Code manipulates raw data structures, bytes, indices. Loops are explicit. Business meaning is inferred from comments. | Code reads like a domain description. Operations are named after business concepts. Implementation details are hidden. |

**Tradeoff:** Low abstraction is easier to optimize and debug. High abstraction is easier to read and maintain but harder to trace through layers.

**How to identify:** Read a function aloud. Does it describe *what* the business wants or *how* the machine achieves it?

---

## code-density

**Definition:** How much logic is packed per line or per function.

| 0 — Sparse | 10 — Dense |
|---|---|
| One operation per line. Functions are short. Variables are named intermediates. Whitespace is generous. | Method chains. Nested comprehensions. Ternary-heavy. Functions are long and do many things. |

**Tradeoff:** Sparse code is easier to step through and review. Dense code is faster to write and may be more expressive to domain experts.

**How to identify:** Look at average function length and nesting depth. Count chained calls. Check if intermediate results are assigned to named variables or composed inline.

---

## generality

**Definition:** How much code is written to handle one specific case vs a class of cases.

| 0 — Specific | 10 — General |
|---|---|
| Each feature is implemented for its exact current requirements. No hooks for future cases. | Code is parameterized, plugin-based, or generic. Features are implemented as instances of a framework. |

**Tradeoff:** Specific code is simpler and faster to write. General code handles future requirements but adds complexity that may never pay off.

**How to identify:** Look for hardcoded assumptions (specific entity types, fixed enum values). Check whether similar features share infrastructure or are duplicated with slight variations.

---

## encapsulation

**Definition:** How well internals are hidden from external callers.

| 0 — Open | 10 — Encapsulated |
|---|---|
| Internal fields, implementation functions, and intermediate state are accessible from outside. Callers reach into objects directly. | Only a narrow public interface is exposed. Internal state changes are mediated. Callers don't know how things work. |

**Tradeoff:** Open code is easier to inspect and patch quickly. Strong encapsulation makes invariants easier to enforce but can make debugging harder.

**How to identify:** Count public vs private fields/methods. Check whether tests access internals directly. Look for reach-through patterns (e.g., `user.profile.settings.theme`).

---

## configuration

**Definition:** How much behavior is controlled by runtime config vs hardcoded at build time.

| 0 — Hardcoded | 10 — Highly Configurable |
|---|---|
| Values and behavior are fixed in source. Changing behavior requires changing and redeploying code. | Nearly all behavior is driven by config files, environment variables, or runtime parameters. |

**Tradeoff:** Hardcoded systems are simpler and have fewer moving parts. High configurability allows operators to tune behavior without redeploys but increases the surface area for misconfiguration.

**How to identify:** Look for magic strings/numbers in business logic. Check how many things live in `.env`, config files, or feature flag systems. Count how many behaviors change per environment.

---

## service-granularity

**Definition:** How finely the system is divided into independently deployable units.

| 0 — Monolith | 10 — Microservices |
|---|---|
| All functionality lives in one deployable unit. Modules are logical boundaries only. | Each domain or capability is a separate deployable service with its own data store. |

**Tradeoff:** Monoliths are simpler to develop, test, and debug. Microservices enable independent scaling and deployment but add operational and communication complexity.

**How to identify:** Count independently deployable artifacts. Check whether cross-domain calls are in-process function calls or network calls. Look for service registries, API gateways, or inter-service auth.

---

## data-consistency

**Definition:** How strongly the system guarantees that data is consistent at all times.

| 0 — Eventual Consistency | 10 — Strong Consistency |
|---|---|
| Data may be temporarily out of sync across nodes or services. Reads may return stale data. Conflicts are resolved later. | All reads reflect the latest committed write. Distributed transactions or strict serializability are used. |

**Tradeoff:** Eventual consistency allows higher throughput and availability. Strong consistency simplifies application logic but limits scalability.

**How to identify:** Look for event sourcing, CQRS patterns, async propagation, or conflict-resolution logic. Check transaction boundaries. Look for `SELECT FOR UPDATE`, two-phase commit, or distributed locking.

---

## synchrony

**Definition:** How much work happens synchronously (blocking) vs asynchronously (non-blocking).

| 0 — Synchronous | 10 — Asynchronous |
|---|---|
| Operations block until complete. Request-response is the dominant pattern. Callers wait for results. | Operations return immediately. Results arrive via callbacks, promises, events, or queues. Work is decoupled in time. |

**Tradeoff:** Synchronous code is easier to reason about and debug. Async code enables higher throughput but introduces ordering, error handling, and debugging complexity.

**How to identify:** Count async/await, promises, callbacks, message queues. Look at whether the API layer waits for downstream calls before responding or fires and forgets.

---

## statefulness

**Definition:** How much application state is held in memory vs reconstructed from storage on each operation.

| 0 — Stateless | 10 — Stateful |
|---|---|
| Each request carries all context. Servers hold no session state. Any instance can handle any request. | Long-lived processes accumulate state in memory. Sessions, caches, or in-memory data structures are critical to operation. |

**Tradeoff:** Stateless services scale horizontally and recover cleanly. Stateful systems can be faster and support richer interaction models but require careful lifecycle management.

**How to identify:** Look for in-memory session stores, sticky sessions, global variables, or singletons that accumulate data over a process lifetime.

---

## test-coverage-posture

**Definition:** How broadly the codebase is covered by automated tests, and what kinds of tests are valued.

| 0 — Minimal Testing | 10 — Comprehensive Testing |
|---|---|
| Tests exist for critical paths only. Coverage is low. Manual QA is the primary safety net. | High coverage across units, integration, and end-to-end. Tests are treated as first-class code. |

**Tradeoff:** Minimal testing is faster in the short term. Comprehensive testing slows initial development but reduces regression risk and supports refactoring.

**How to identify:** Check coverage metrics if available. Count test files vs source files. Look at the ratio of unit tests to integration tests. Check whether tests are skipped, commented out, or written after the fact.

---

## documentation-density

**Definition:** How much inline and external documentation exists relative to code volume.

| 0 — Minimal Docs | 10 — Heavily Documented |
|---|---|
| Code is expected to be self-documenting. Comments are rare and reserved for non-obvious logic. No API docs. | Every public interface has docstrings. Architecture decisions are recorded. Inline comments explain intent. |

**Tradeoff:** Minimal docs reduce maintenance overhead. Heavy documentation helps onboarding and cross-team collaboration but goes stale.

**How to identify:** Check the ratio of comment lines to code lines. Look for docstrings on public functions. Check for README, ADR, or architecture docs.

---

## type-strictness

**Definition:** How strongly the type system is used to encode invariants and prevent invalid states.

| 0 — Weakly Typed / Untyped | 10 — Strictly Typed |
|---|---|
| `any`, `object`, or dynamic types are common. Type annotations are optional and often absent. Runtime errors are the primary discovery mechanism. | Types encode domain invariants. Nullable types are explicit. Impossible states are unrepresentable. `any` is banned. |

**Tradeoff:** Loose types are faster to prototype. Strict types catch errors at compile time and serve as machine-checked documentation, but require investment.

**How to identify:** Look for `any`, untyped function parameters, missing return types, or `@ts-ignore`. Check whether nullable values are explicitly typed or assumed non-null.

---

## error-handling-loudness

**Definition:** How visibly errors are surfaced vs swallowed.

| 0 — Silent | 10 — Loud |
|---|---|
| Errors are caught and suppressed. Operations fail silently. Empty catch blocks. Fallback to defaults on failure. | Errors propagate. Unhandled failures crash or produce clear logs. Operations fail fast with explicit messages. |

**Tradeoff:** Silent failures prevent user-facing errors but hide bugs. Loud failures surface problems quickly but may be disruptive in production.

**How to identify:** Search for empty catch blocks, `catch (e) {}`, swallowed promise rejections, or functions that return `null`/`undefined` on error without logging. Check whether errors are logged with context or rethrown.

---

## defensive-programming

**Definition:** How much the code guards against invalid inputs, unexpected states, and external failures.

| 0 — Trusting | 10 — Defensive |
|---|---|
| Inputs are assumed valid. External calls are assumed to succeed. Assertions are absent. | Inputs are validated at every boundary. Preconditions are asserted. External calls have fallbacks, retries, and timeouts. |

**Tradeoff:** Trusting code is simpler and less verbose. Defensive code is safer but adds boilerplate and can obscure the happy path.

**How to identify:** Look for input validation, assertion libraries, guard clauses, null checks, and explicit handling of network/timeout failures.

---

## memory-speed-tradeoff

**Definition:** Whether the codebase prefers to compute results or cache/precompute them.

| 0 — Compute-Heavy | 10 — Cache-Heavy |
|---|---|
| Results are computed on demand. No caching. Memory usage is minimal. Latency is paid per request. | Results are precomputed and stored. Caches are aggressively used. Memory is traded for speed. |

**Tradeoff:** Compute-heavy systems are simpler and always fresh. Cache-heavy systems are faster but require cache invalidation logic and can serve stale data.

**How to identify:** Look for memoization, Redis/memcached usage, precomputed aggregates, or materialized views. Check whether expensive operations are repeated per request.

---

## build-vs-buy-appetite

**Definition:** Whether the team prefers to build custom solutions or adopt external libraries and services.

| 0 — Build Everything | 10 — Buy/Adopt Everything |
|---|---|
| Custom implementations for most infrastructure concerns. Few external dependencies. | External libraries and SaaS services are the default. New requirements start with "what exists?" |

**Tradeoff:** Building gives control and avoids dependency risk. Buying accelerates development but introduces vendor lock-in and external failure modes.

**How to identify:** Count external dependencies in package manifests. Look for custom implementations of things that have well-known libraries (auth, queues, metrics). Check the ratio of in-house infrastructure to third-party services.

---

## dependency-appetite

**Definition:** How willing the project is to add new external dependencies.

| 0 — Dependency-Averse | 10 — Dependency-Friendly |
|---|---|
| Dependencies are added rarely and only for critical needs. Vendored or pinned tightly. The team prefers fewer moving parts. | New libraries are added freely. Package count is high. The team trusts the ecosystem. |

**Tradeoff:** Dependency-averse projects are more predictable and have a smaller attack surface. Dependency-friendly projects move faster but face version conflicts, security exposure, and breakage from upstream changes.

**How to identify:** Count total dependencies in package manifests. Check whether dependencies are pinned exactly or with ranges. Look at how often the lock file changes. Check for `.vendor` directories or in-tree copies of libraries.
