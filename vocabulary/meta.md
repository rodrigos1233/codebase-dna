# Meta Concepts

These concepts apply across all axes and properties. They are not additional axes — they are qualifiers and alerts that modify how fingerprint entries should be read and written.

---

## Descriptive vs Aspirational

Every position in a `dna.md` is one of two things:

**Descriptive** — reflects what the codebase currently does, based on evidence from reading the code. This is the default. If a position has no marker, it is descriptive.

**Aspirational** — reflects what the team intends the codebase to become, even though the code doesn't reflect it yet. This must be explicitly marked with `aspirational: true` in the template.

### Why the distinction matters

An agent using a `dna.md` to write new code must know whether to *match* the existing pattern or *lead toward* the intended pattern. These are opposite behaviors:

- A codebase described as `test-coverage-posture: 2` (descriptive) means most of the code has no tests. A new function should have tests anyway — but the agent shouldn't be surprised to find untested neighbors.
- A codebase described as `test-coverage-posture: 2` with `aspirational: 8` means the team wants comprehensive testing. New code should aim for the aspirational level, not match the current state.

### How to write aspirational positions

```yaml
test-coverage-posture:
  position: 2          # current reality
  aspirational: 8      # where it should go
  notes: >
    Most legacy modules have no tests. New modules should have unit and
    integration tests. Do not imitate the test-free pattern even where it
    is common.
```

### How to read aspirational positions

When both `position` and `aspirational` are present:
- Use `aspirational` as your target when writing new code.
- Use `position` to calibrate expectations about what you'll find in existing code.
- Do not propagate the existing pattern just because it is common.

---

## Dead Ends

A dead end is a pattern that was tried, caused problems, and was intentionally abandoned. Dead ends are recorded so that future agents and developers do not rediscover the same wrong path.

### What to record

- The pattern that was tried
- Why it was abandoned (the actual problem, not vague dissatisfaction)
- What replaced it, if anything

### Format

```yaml
dead-ends:
  - pattern: "Using class-based React components with local state for data fetching"
    reason: >
      Led to tangled lifecycle logic and race conditions. Superseded by
      React Query + functional components.
    replaced-by: "React Query hooks in functional components"

  - pattern: "Storing user sessions in JWT with 7-day expiry and no revocation"
    reason: >
      No way to force logout on credential compromise. Replaced with
      short-lived JWTs and server-side refresh token store.
    replaced-by: "15-min JWT + Redis-backed refresh tokens"
```

### How agents should use dead ends

Before proposing a solution, check whether it matches a recorded dead end. If it does, do not propose it — even if it seems like the most natural approach. Explain why the dead end applies if it's non-obvious.

---

## Simplicity Zones

A simplicity zone is an area of the codebase that is intentionally kept simple and must not be generalized, abstracted, or made "better" in ways that add complexity.

### Why simplicity zones exist

Some areas are simple for a reason:
- Performance-critical paths where overhead matters
- Interfaces used by non-engineers (scripts, config files, CLI tools)
- Bootstrapping code that runs before the framework is initialized
- Areas where the cost of a bug is extremely high

Agents default toward good engineering: abstraction, reuse, DRY. In simplicity zones, that default is wrong.

### Format

```yaml
simplicity-zones:
  - area: "Database migration scripts"
    rule: >
      Migrations must be plain SQL or minimal ORM calls. No helper
      functions, no shared utilities, no imports from application code.
      Each migration must be fully self-contained and readable in isolation.

  - area: "Health check endpoint"
    rule: >
      Must remain a single function with no dependencies. Do not add
      middleware, auth, or logging. It is called by load balancers under
      pressure and must never fail due to application-layer issues.
```

### How agents should use simplicity zones

When working in or near a simplicity zone, match the existing simplicity. Do not apply standard engineering improvements. If a task requires adding complexity to a simplicity zone, flag it explicitly before proceeding.

---

## Drift

Drift occurs when observed code contradicts the stated fingerprint. It is neither a bug nor a violation — it is information. Drift means either:

1. The fingerprint is outdated and needs updating, or
2. The code is drifting away from the team's stated intent

### When to flag drift

Flag drift when:
- You observe a pattern in the code that contradicts a recorded position (e.g., the fingerprint says `pure-functions: true` but you find stateful logic throughout a module)
- You are asked to write code and the relevant axis gives contradictory signals (the template says one thing, what you observe says another)
- A dead end pattern appears in active use in a significant portion of the codebase

### How to report drift

Do not silently pick one signal over the other. Explicitly surface the contradiction:

```
Drift detected: The dna.md records `error-handling-loudness: 8` (loud, fail-fast),
but the `payments/` module has 14 empty catch blocks and no logging. This may mean
the fingerprint is aspirational in this area, or that this module was written before
the standard was established.

Proceeding with the loud pattern as instructed by the fingerprint. Flagging for review.
```

### Updating the fingerprint after drift

If drift is widespread and represents a real change in direction, update the fingerprint rather than treating all existing code as violations. The fingerprint should reflect reality, with aspirational positions used for forward guidance.
