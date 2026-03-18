# Sampling Strategy

A full codebase read is rarely possible or necessary. This document specifies which files to read, in what order, and what to extract from each — so an audit can produce a reliable fingerprint in a single focused pass.

---

## Read Order

### 1. Project Root (5 minutes)

Start here before reading any code. These files reveal architecture intent and philosophy.

**Read:**
- `README.md` — stated goals, setup instructions, technology choices
- `package.json` / `go.mod` / `Cargo.toml` / `pyproject.toml` — dependency count, test framework, build tools
- `tsconfig.json` / `.eslintrc` / `pyproject.toml` / `clippy.toml` — type strictness, lint rules, code style enforcement
- `.env.example` or `config/` directory — how much behavior is configurable
- `docker-compose.yml` / `Makefile` / `justfile` — how the system is run and what services it depends on
- `CLAUDE.md` / `AGENTS.md` / `.cursorrules` — any existing style guidance the team has written for agents

**Extract:**
- Technology stack and version constraints
- External services the system depends on
- Existing style documentation (already partially fills the fingerprint)
- How many dependencies exist (dependency-appetite signal)

---

### 2. Entry Points (10 minutes)

Entry points reveal how the system is organized and what it values at its boundaries.

**For web services:** `main.go`, `app.py`, `server.ts`, `index.ts`, or equivalent
**For CLIs:** the top-level command definition
**For libraries:** the public export file
**For jobs:** the job runner or scheduler entrypoint

**Read:**
- The main entrypoint file
- The router or request dispatcher (e.g., `routes.ts`, `router.go`)
- The middleware stack — what runs before every request

**Extract:**
- Is there a clear auth boundary in middleware? (authentication-boundaries)
- How are errors handled at the top level? (error-handling-loudness)
- Sync or async request handling? (synchrony)
- How is configuration loaded? (configuration)

---

### 3. One Full Feature Slice (20 minutes)

Pick one representative feature — not the simplest and not the most complex. Trace it from the inbound request to the data layer and back.

**Ideal feature:** A CRUD operation on a core domain entity (not auth, not health check).

**Files to read in order:**
1. The route or handler file that receives the request
2. Any validation or serialization layer
3. The service or use-case file that contains business logic
4. The data access layer (repository, ORM model, raw queries)
5. Any shared utilities or helpers called along the way

**Extract:**
- coupling: Do layers import each other directly or through interfaces?
- abstraction-level: Does the service layer read like the business domain?
- code-density: How long are functions? Are they chained or stepped?
- pure-functions: Does the service layer take inputs and return outputs, or read/write state internally?
- error-handling-loudness: Do errors propagate or get swallowed?
- type-strictness: Are all types explicit? Is `any` used?
- null-safety: Are nulls handled explicitly?

---

### 4. Test Files (10 minutes)

Read 2–3 test files for the feature slice you already traced, plus any test utility files.

**Look for:**
- Test file to source file ratio (test-coverage-posture signal)
- Unit tests vs integration tests vs end-to-end — which is dominant?
- Whether tests use mocks/spies or real dependencies (coupling and defensive-programming signal)
- Whether test data is hardcoded or generated (determinism signal)
- Whether tests read as specifications or as coverage exercises

**Extract:**
- test-coverage-posture: High, low, or spotty?
- What the team values: fast unit tests, integration confidence, or end-to-end coverage?
- Whether tests are first-class code or afterthoughts (naming, organization, comments)

---

### 5. Config and Infrastructure Files (5 minutes)

**Read:**
- All files in `config/`, `infra/`, `terraform/`, or `k8s/`
- `.github/workflows/` or CI configuration
- Any secrets management configuration

**Extract:**
- Are secrets in environment variables, a vault, or (flag this) in code?
- Is disk-level or database-level encryption configured? (encryption-at-rest)
- What environments exist (dev/staging/prod)? How different are they?
- Is deployment automated? What does the release process look like?

---

## Large Codebases

When the codebase is too large for the above pass (>50k lines), apply these constraints:

1. **Limit the feature slice to one layer.** If you can only read one file, read the service/use-case layer. It reveals the most about business logic patterns.

2. **Sample horizontally, not vertically.** Read the first 50 lines of 10 different service files rather than all of 2. Consistency of style matters more than depth for fingerprinting.

3. **Weight recent files.** Check `git log --since="90 days ago" --name-only` to identify which files are actively worked on. Sample from active files — they reflect current intent, not legacy patterns.

4. **Note what you didn't read.** If large areas of the codebase were not sampled, record this as uncertainty in the fingerprint notes.

---

## Red Flags That Suggest Aspirational Overrides Are Needed

These observations suggest the descriptive fingerprint should be accompanied by aspirational positions:

- **Inconsistent patterns across modules** — some modules are well-structured, others are not. The team likely has a direction but hasn't refactored everything.
- **A `CLAUDE.md` or `AGENTS.md` that contradicts what the code does** — the team's stated intent differs from current reality. Use the stated intent as aspirational.
- **Active refactoring in progress** — if recent commits show a pattern being replaced (e.g., converting callbacks to async/await), record the target as aspirational.
- **A `TODO` or `FIXME` density well above zero** — technical debt is acknowledged. The descriptive position reflects reality; aspirational should reflect intent.
- **Tests that are skipped or commented out at a high rate** — test-coverage-posture should have an aspirational value if skipped tests suggest the team wants more coverage.
- **Dead end patterns still in use in old modules** — record both the dead end and the fact that it exists in the codebase for historical reasons, not as a model to follow.
