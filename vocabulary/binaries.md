# Binaries

Binary properties have exactly two valid values: `true` or `false`. Unlike sliders, there is no spectrum — the codebase either has the property or it doesn't.

Every binary has an optional `scope:` field. Use it when the property is true in some layers and false in others. For most binaries, the scope is the most interesting part — a codebase where pure functions apply only to the utility layer behaves very differently from one where they apply throughout.

---

## pure-functions

**Definition:** A function is pure if it always returns the same output for the same input and produces no side effects (no I/O, no mutation of external state, no randomness). This property is `true` if pure functions are the dominant pattern in the codebase, not just present in some utilities.

**Scope guidance:** Almost always worth scoping. Pure functions are common in utilities but rare in service layers. Recording `scope: "utility and transformation layers"` tells an agent more than a bare `true`.

**How to verify:**
- Look at the most commonly written functions. Do they take inputs and return outputs, or do they read/write state?
- Check for functions that accept no parameters but return data (implicit global reads).
- Look for functions that mutate their arguments or module-level variables.
- Check whether the test style favors input/output assertions or spy/mock assertions on side effects.

**Why it matters for code generation:** In a pure-function codebase, new logic should be written as transformations: take data in, return data out. Side effects are isolated at boundaries. An agent that writes stateful, mutating functions will produce code that doesn't fit.

---

## idempotency

**Definition:** An operation is idempotent if calling it multiple times with the same inputs produces the same result as calling it once. This property is `true` if the codebase consistently designs operations — especially writes, mutations, and API endpoints — to be safe to retry.

**Scope guidance:** Scoping is often necessary. Write endpoints and background jobs are typically in scope; read endpoints are trivially idempotent and don't need to be recorded. Example: `scope: "write endpoints and background jobs"`.

**How to verify:**
- Check PUT/PATCH endpoint semantics: do they replace state or accumulate it?
- Look for upsert patterns vs insert-only patterns.
- Check background jobs and event handlers: are they designed to handle duplicate delivery?
- Look for idempotency keys in payment processing or external API calls.
- Check whether database migrations are idempotent.

**Why it matters for code generation:** In an idempotent codebase, new operations should be written defensively: "what happens if this runs twice?" is a required question. An agent that writes non-idempotent operations may introduce data corruption bugs that are hard to trace.

---

## null-safety

**Definition:** Null/undefined values are explicitly represented in the type system and callers are required to handle them. This property is `true` if the codebase treats null as a typed possibility rather than an implicit option for all values.

**Scope guidance:** Useful when a project mixes typed and untyped areas (e.g., strict TypeScript on the server, loose JavaScript in legacy frontend). Example: `scope: "server-side TypeScript only"`.

**How to verify:**
- In TypeScript: check whether `strictNullChecks` is enabled in `tsconfig.json`.
- Look for optional chaining (`?.`) vs direct property access on values that could be null.
- Check whether functions that may return nothing return `T | null`, `Option<T>`, or `Maybe<T>` — vs returning `T` and silently returning `null` or `undefined`.
- Look for null guard patterns at function entry points.

**Why it matters for code generation:** In a null-safe codebase, every function that might not return a value must return a typed optional, and every caller must handle the null case. An agent that omits null handling will either fail type checks or introduce runtime null dereferences.

---

## determinism

**Definition:** Given the same inputs and environment, the system produces the same outputs. This property is `true` if the codebase avoids hidden sources of non-determinism in its core logic (randomness, time, concurrency, environment-dependent behavior).

**Scope guidance:** Useful when determinism is enforced in the domain layer but not at the infrastructure layer. Example: `scope: "domain and service layers — infrastructure layer explicitly excluded"`.

**How to verify:**
- Look for uses of `Math.random()`, `Date.now()`, `uuid()`, or `crypto.randomBytes()` inside business logic vs at system boundaries.
- Check whether time is injected as a parameter or read from a global clock inside functions.
- Look for race conditions in concurrent code.
- Check whether tests are deterministic (no flaky test history, no time-dependent assertions).

**Why it matters for code generation:** In a deterministic codebase, non-deterministic inputs (time, randomness, IDs) should be injected from outside, not generated inside logic functions. An agent that embeds `Date.now()` inside a pure calculation will break testability and reproducibility.

---

## encryption-at-rest

**Definition:** Sensitive data is encrypted before being written to durable storage. This property is `true` if the codebase consistently encrypts sensitive fields, databases, or file stores — not just at the infrastructure level but as an explicit application-layer concern.

**Scope guidance:** Almost always worth scoping to the specific fields or storage layers covered. Example: `scope: "PII fields (email, phone) encrypted at column level — other data covered by disk encryption only"`.

**How to verify:**
- Check database column definitions for sensitive fields (PII, secrets, tokens). Are they stored as plaintext or encrypted?
- Look for encryption/decryption logic in model or repository layers.
- Check whether secrets management (API keys, passwords) uses a vault or secrets manager vs plaintext env vars checked into config.
- Look for disk-level encryption configuration in infrastructure code (Terraform, Helm, Kubernetes).

**Why it matters for code generation:** In a codebase that encrypts at rest, new sensitive fields must go through the encryption layer. An agent that adds a `password` column without encryption will introduce a compliance violation. The scope field should specify which fields are sensitive and what mechanism is used.

---

## authentication-boundaries

**Definition:** Authentication is checked at clearly defined and consistently enforced entry points, not scattered throughout business logic. This property is `true` if there is a single, auditable authentication layer that all inbound requests pass through before reaching business code.

**Scope guidance:** Useful when different request types have different auth mechanisms. Example: `scope: "HTTP API — internal gRPC service calls use mTLS separately"`.

**How to verify:**
- Look for authentication middleware or guards. Are they applied globally or per-route?
- Check whether business logic functions ever perform their own auth checks or assume the caller has already verified identity.
- Look for routes or endpoints that bypass authentication (intentionally or not).
- Check whether internal service-to-service calls have their own auth mechanism.

**Why it matters for code generation:** In a codebase with clear authentication boundaries, new endpoints must be registered behind the auth layer. An agent that adds a route without applying the auth middleware will create an unauthenticated endpoint. The scope field should describe the authentication mechanism and where to apply it.
