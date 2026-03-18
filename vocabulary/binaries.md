# Binaries

Binary properties have exactly two valid values: `true` or `false`. Unlike sliders, there is no spectrum — the codebase either has the property or it doesn't.

Some binary properties can be scoped (e.g., "pure functions: true for utility layer, false for service layer"). When scope matters, record it in the `notes:` field rather than forcing a single value.

---

## pure-functions

**Definition:** A function is pure if it always returns the same output for the same input and produces no side effects (no I/O, no mutation of external state, no randomness). This property is `true` if pure functions are the dominant pattern in the codebase, not just present in some utilities.

**How to verify:**
- Look at the most commonly written functions. Do they take inputs and return outputs, or do they read/write state?
- Check for functions that accept no parameters but return data (implicit global reads).
- Look for functions that mutate their arguments or module-level variables.
- Check whether the test style favors input/output assertions or spy/mock assertions on side effects.

**Why it matters for code generation:** In a pure-function codebase, new logic should be written as transformations: take data in, return data out. Side effects are isolated at boundaries. An agent that writes stateful, mutating functions will produce code that doesn't fit.

---

## idempotency

**Definition:** An operation is idempotent if calling it multiple times with the same inputs produces the same result as calling it once. This property is `true` if the codebase consistently designs operations — especially writes, mutations, and API endpoints — to be safe to retry.

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

**How to verify:**
- In TypeScript: check whether `strictNullChecks` is enabled in `tsconfig.json`.
- Look for optional chaining (`?.`) vs direct property access on values that could be null.
- Check whether functions that may return nothing return `T | null`, `Option<T>`, or `Maybe<T>` — vs returning `T` and silently returning `null` or `undefined`.
- Look for null guard patterns at function entry points.

**Why it matters for code generation:** In a null-safe codebase, every function that might not return a value must return a typed optional, and every caller must handle the null case. An agent that omits null handling will either fail type checks or introduce runtime null dereferences.

---

## determinism

**Definition:** Given the same inputs and environment, the system produces the same outputs. This property is `true` if the codebase avoids hidden sources of non-determinism in its core logic (randomness, time, concurrency, environment-dependent behavior).

**How to verify:**
- Look for uses of `Math.random()`, `Date.now()`, `uuid()`, or `crypto.randomBytes()` inside business logic vs at system boundaries.
- Check whether time is injected as a parameter or read from a global clock inside functions.
- Look for race conditions in concurrent code.
- Check whether tests are deterministic (no flaky test history, no time-dependent assertions).

**Why it matters for code generation:** In a deterministic codebase, non-deterministic inputs (time, randomness, IDs) should be injected from outside, not generated inside logic functions. An agent that embeds `Date.now()` inside a pure calculation will break testability and reproducibility.

---

## encryption-at-rest

**Definition:** Sensitive data is encrypted before being written to durable storage. This property is `true` if the codebase consistently encrypts sensitive fields, databases, or file stores — not just at the infrastructure level but as an explicit application-layer concern.

**How to verify:**
- Check database column definitions for sensitive fields (PII, secrets, tokens). Are they stored as plaintext or encrypted?
- Look for encryption/decryption logic in model or repository layers.
- Check whether secrets management (API keys, passwords) uses a vault or secrets manager vs plaintext env vars checked into config.
- Look for disk-level encryption configuration in infrastructure code (Terraform, Helm, Kubernetes).

**Why it matters for code generation:** In a codebase that encrypts at rest, new sensitive fields must go through the encryption layer. An agent that adds a `password` column without encryption will introduce a compliance violation. The notes field should specify which fields are sensitive and what mechanism is used.

---

## authentication-boundaries

**Definition:** Authentication is checked at clearly defined and consistently enforced entry points, not scattered throughout business logic. This property is `true` if there is a single, auditable authentication layer that all inbound requests pass through before reaching business code.

**How to verify:**
- Look for authentication middleware or guards. Are they applied globally or per-route?
- Check whether business logic functions ever perform their own auth checks or assume the caller has already verified identity.
- Look for routes or endpoints that bypass authentication (intentionally or not).
- Check whether internal service-to-service calls have their own auth mechanism.

**Why it matters for code generation:** In a codebase with clear authentication boundaries, new endpoints must be registered behind the auth layer. An agent that adds a route without applying the auth middleware will create an unauthenticated endpoint. The notes field should describe the authentication mechanism and how to apply it to new code.
