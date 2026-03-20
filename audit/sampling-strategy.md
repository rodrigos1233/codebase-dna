# Sampling Strategy

The reading goal is not to classify the codebase across a set of axes. It is to find information an agent cannot see by reading a single file: structural constraints, established conventions that required deliberate choices, and traps.

Read in order. Stop a section early if it stops producing new non-local signal.

---

## 1. Existing agent and architecture documentation (5 minutes)

Read these first. They often contain explicit conventions, stated dead ends, and placement rules that would otherwise require inference.

**Read:**
- `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, or equivalent
- `README.md` — stated constraints, not just project description
- Any `decisions/`, `adr/`, or `docs/architecture/` files
- Inline comments that explain *why* — not what

**Looking for:**
- Explicitly stated conventions (these become `documented` evidence entries)
- Stated dead ends or rejected approaches
- Known-debt acknowledgements
- Simplicity rules ("do not add X to Y")

---

## 2. Configuration and enforcement files (5 minutes)

These reveal which constraints are mechanically enforced vs convention-only.

**Read:**
- `package.json` / `go.mod` / `Cargo.toml` — library choices (stack)
- `tsconfig.json`, `.eslintrc`, `pyproject.toml`, `clippy.toml` — type strictness, lint rules
- `eslint-plugin-import` or similar boundary enforcement configs
- `.env.example` or `config/` — configurable vs hardcoded behavior
- `docker-compose.yml` / `Makefile` — what services exist, how they communicate

**Looking for:**
- Which lint rules enforce boundaries (boundaries section)
- Strict type settings — `strictNullChecks`, `noImplicitAny` (informs patterns and known-debt)
- Library choices that reveal project opinions (stack)
- Whether auth or validation enforcement is in config or convention

---

## 3. Entry points and module boundary definitions (10 minutes)

These reveal how the system is organized and where structural constraints live.

**Read:**
- The main entrypoint: `main.go`, `server.ts`, `app.py`, `index.ts`
- The router or request dispatcher: where routes are registered
- The middleware stack: what runs before every request, in what order
- Any `index.ts` / `mod.rs` / `__init__.py` that define module public interfaces

**Looking for:**
- Where authentication is checked — and what comes before vs after it (boundaries)
- The dependency direction between modules — what imports what (boundaries)
- Whether the router enforces placement or relies on convention (boundaries, evidence level)
- What the public interface of each module exposes — what is intentionally hidden (boundaries)

---

## 4. One full feature slice — architecture read, not classification read (15 minutes)

Trace one representative feature from inbound request to data layer. Pick a feature that touches multiple layers — not the simplest (auth, health check) and not the most complex.

**Files to read in order:**
1. Route or handler
2. Validation or serialization layer (if separate)
3. Service or use-case layer
4. Data access layer

**Looking for — non-local concerns only:**
- Does the service layer call the data layer directly, or through an interface? (boundaries)
- Is there a consistent shape at each layer boundary — DTOs, domain types, persistence types? (state-contracts or patterns)
- Where does transformation between shapes happen, and is that placement consistent? (patterns)
- Are non-deterministic inputs (time, random, UUIDs) injected or generated inline? (patterns)
- Is there shared mutable state that must be updated in multiple places together? (state-contracts)

Do not spend time classifying things visible from the file itself: naming conventions, function length, comment density, type annotation coverage. These are locally legible.

---

## 5. Git history — dead end evidence (5 minutes)

Dead ends require evidence. The git log is the primary source.

**Run:**
```
git log --oneline --since="18 months ago" | head -60
git log --oneline --all --grep="revert\|remove\|replace\|abandon\|migrate" | head -30
```

**Looking for:**
- Commits that remove or replace a pattern ("replace class components with hooks", "remove redux-saga, use react-query")
- Revert commits that explain why an approach was abandoned
- Large refactors that signal a direction change

If you find a dead end signal in git, read 2–3 commits to confirm the pattern is actually gone from new code, not just renamed.

---

## 5b. Known-debt signal scan (5 minutes)

Before finalising `known-debt`, scan all files in the sampled scope for inline comments containing `race`, `caveat`, `warning`, `note:`, `TODO`, or `do not`. These are high-probability known-debt signals that may not surface in architectural files or CLAUDE.md.

```
grep -rn --include="*.ts" --include="*.go" --include="*.py" -i "race\|caveat\|warning\|note:\|TODO\|do not" <src-dir>
```

For each match, check whether the surrounding comment names a specific file or location and says what to do instead. If it does, it is a candidate known-debt entry. If it is a general warning about a language or library feature with no codebase-specific constraint, omit it.

---

## 6. Test files (10 minutes)

Read 2–3 test files for the feature slice you traced, plus any test utility or fixture files.

**Looking for — non-local concerns only:**
- Are tests organized to reflect layer boundaries, or do they cross layers freely? (boundaries signal)
- Do test utilities enforce a particular test data pattern? (patterns)
- Do tests mock internal modules — which modules are considered internal boundaries? (boundaries)
- Are there test helpers that wrap a particular state setup — revealing state-contracts?
- Do tests skip or are they commented out in large numbers? (if so, note in audit-notes)

---

## Large codebases (>50k lines)

Apply these constraints when a full pass is not possible:

1. **Weight recent files.** Run `git log --since="90 days ago" --name-only --pretty=format: | sort | uniq -c | sort -rn | head -30` to find the most actively modified files. Sample from those — they reflect current intent.

2. **Sample module boundaries horizontally.** Read the first 30 lines of 8–10 different module index files rather than tracing one full feature slice. Consistency of the boundary pattern matters more than depth.

3. **Prioritize agent docs.** A good CLAUDE.md or AGENTS.md for a large codebase is worth 30 minutes of code reading. Read it first and completely.

4. **Record what you did not read.** If large areas were not sampled, note this in `audit-notes`. An `inferred` entry is better than a false `observed` entry.

---

## What not to look for

Do not spend reading time on things that are locally legible. An agent will see these when working in context:

- Naming conventions and code style
- Function length and comment density
- Type annotation coverage in individual files
- Error handling patterns visible in the file being modified
- Which libraries are imported (visible from the file)

Record these only if the pattern is inconsistent across the codebase and the inconsistency itself is non-obvious signal (for example, a known-debt entry explaining why one module uses a different pattern than the rest).
