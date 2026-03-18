# codebase-dna

A Claude Code skill that gives agents a shared vocabulary and tooling for describing, auditing, and using codebase style fingerprints.

## What it does

`codebase-dna` lets you capture *how* a codebase is written — not what it does — in a structured file called `dna.md`. Once you have one, agents can write new code that fits the project's style, patterns, and constraints without reading the whole codebase first.

Three use cases:

- **Audit** — run an agent over a codebase to produce a draft `dna.md`
- **Write** — give an agent a `dna.md` and it writes consistent code without reading everything
- **Fill manually** — use the vocabulary and templates to write or correct a fingerprint yourself

---

## Installation

### Claude Code

Skills live in `~/.claude/skills/`. Clone this repo there (or copy the directory):

```bash
git clone https://github.com/your-org/codebase-dna ~/.claude/skills/codebase-dna
```

Or if you want to keep the repo elsewhere and symlink it:

```bash
git clone https://github.com/your-org/codebase-dna ~/path/to/codebase-dna
ln -s ~/path/to/codebase-dna ~/.claude/skills/codebase-dna
```

Claude Code discovers skills at startup. Restart any running session after installing.

**Verify:** Start a new session and ask Claude to audit a codebase or produce a `dna.md`. It should invoke this skill automatically.

---

### Codex

Codex discovers skills from `~/.agents/skills/`. Clone and symlink:

```bash
git clone https://github.com/your-org/codebase-dna ~/.codex/codebase-dna
mkdir -p ~/.agents/skills
ln -s ~/.codex/codebase-dna ~/.agents/skills/codebase-dna
```

Restart Codex after installing.

**Windows (PowerShell):**

```powershell
git clone https://github.com/your-org/codebase-dna "$env:USERPROFILE\.codex\codebase-dna"
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
cmd /c mklink /J "$env:USERPROFILE\.agents\skills\codebase-dna" "$env:USERPROFILE\.codex\codebase-dna"
```

**Verify:**

```bash
ls -la ~/.agents/skills/codebase-dna
```

---

### Manual (any agent)

If your agent supports loading context files directly, copy or reference `SKILL.md` and the relevant subdirectories. The minimum needed per use case:

| Task | Files to include |
|---|---|
| Focused implementation task | `SKILL.md` + fresh/generated `dna.inject.md` |
| Broad or cross-cutting work | `SKILL.md` + your project's `dna.md` |
| Auditing a codebase | `SKILL.md` + `vocabulary/` + `audit/` + `template/dna.template.md` |
| Filling a fingerprint manually | `SKILL.md` + `vocabulary/` + `template/` |

For system prompt injection (e.g. API usage), include the content of the relevant files directly. Use `template/dna.minimal.template.md` as the reusable, prompt-sized fingerprint. Use `template/dna.inject.md` as the format for generating a fresh one-off `dna.inject.md` for a scoped task.

### Validation Rubric

Use the smallest context that still covers the task:

| Pressure scenario | Minimum context to load |
|---|---|
| Audit a codebase from scratch | `SKILL.md` + `vocabulary/` + `audit/` + `template/dna.template.md` |
| Write a focused feature against an existing fingerprint | `SKILL.md` + fresh/generated `dna.inject.md` + only the project files needed for the feature |
| Generate a task-scoped `dna.inject.md` for a narrow bug fix | `SKILL.md` + `scope/prompt.md` + the existing `dna.md` + `template/dna.inject.md` + the task description in prose + only the bug-relevant source files |

Practical checks:

- Do not load audit vocabulary during normal write tasks unless a term is ambiguous.
- Keep `dna.inject.md` in the compact range, not the full fingerprint range.
- Reserve full `dna.md` for onboarding, audit, or broad cross-cutting work.

---

## Updating

```bash
# If installed via git clone:
cd ~/.claude/skills/codebase-dna && git pull

# Codex:
cd ~/.codex/codebase-dna && git pull
```

Skills update instantly through symlinks — no restart needed for updates.

---

## Uninstalling

```bash
# Claude Code:
rm -rf ~/.claude/skills/codebase-dna

# Codex (symlink):
rm ~/.agents/skills/codebase-dna
rm -rf ~/.codex/codebase-dna   # optional, removes the clone
```

---

## Usage

### Audit a codebase

In a new session, tell your agent:

```
Audit this codebase and produce a dna.md fingerprint. Use the codebase-dna skill.
```

The agent will load the vocabulary and audit instructions, read a representative sample of the codebase, and output a filled `dna.md` using the full template.

### Use an existing fingerprint

For broad or cross-cutting work, reference the project fingerprint in your project's `CLAUDE.md` (or equivalent):

```markdown
@dna.md
```

Or say it directly:

```
Use the project fingerprint in dna.md to guide broad or cross-cutting work.
```

For broad or cross-cutting work, the agent will load the fingerprint and use it to match existing patterns when writing new code. If it encounters an unfamiliar term, it will load only the relevant vocabulary section — not the full skill.

For focused implementation tasks, generate a fresh `dna.inject.md` for that task instead of relying on a persistent include.

### Scope a fingerprint to a specific task

Use the narrowest fingerprint that still fits the task:

- Focused implementation task: generate and use `dna.inject.md`.
- Broad or cross-cutting work: load the full `dna.md`.
- Audit or manual fingerprinting: load the vocabulary and template files; add audit materials only when auditing code.

A task-scoped injection is the default for focused implementation work because it keeps only the rules that matter for that change. Generate it like this:

```
Given this task: [describe the task]
And this fingerprint: @dna.md
Run the codebase-dna scope agent and produce a dna.inject.md.
```

The scope agent filters the full fingerprint down to the rules most likely to prevent mistakes for that specific task — typically 20–40 lines. Dead ends, simplicity zones, and known debt always survive. Slider positions and descriptive prose get dropped. The result can be pasted directly into a task-specific system prompt.

### Fill a fingerprint manually

Copy `template/dna.template.md` (full) or `template/dna.minimal.template.md` (for a persistent prompt include) to your project root and fill it in. `template/dna.inject.md` defines the task-scoped injection format; generate a fresh `dna.inject.md` for focused implementation work. The `vocabulary/` files define the axes and properties precisely; use `audit/` only if you also need the sampling strategy or audit prompt.

---

## File structure

```
codebase-dna/
├── SKILL.md                    # Entry point — agents load this first
├── vocabulary/
│   ├── sliders.md              # 19 spectrum axes (coupling, abstraction, etc.)
│   ├── binaries.md             # 6 yes/no properties (pure functions, idempotency, etc.)
│   └── meta.md                 # Descriptive vs aspirational, dead ends, simplicity zones, drift
├── audit/
│   ├── prompt.md               # Ready-to-use prompt for an audit agent
│   └── sampling-strategy.md    # Which files to read and in what order
├── scope/
│   └── prompt.md               # Agent that scopes dna.md to a specific task
└── template/
    ├── dna.template.md         # Full fingerprint template
    ├── dna.minimal.template.md # Compact long-lived fingerprint for reusable prompt injection
    └── dna.inject.md           # Task-scoped injection format/template used by the scope agent
```

### When to use which format

| Format | Use when |
|---|---|
| `dna.template.md` | Auditing, human review, storing in the repo |
| `dna.minimal.template.md` | Compact long-lived fingerprint for a persistent include when you want a slim, stable reference |
| `dna.inject.md` | Per-task context injection for focused implementation work - generate fresh each time |

Use `dna.minimal.template.md` when you want a compact fingerprint that can live in a long-lived prompt include. Use a freshly generated `dna.inject.md` when the task is scoped and you want the narrowest set of rules for that one change. Keep `dna.template.md` for audit/review workflows and for fields that are useful to humans but not worth carrying into injection.
