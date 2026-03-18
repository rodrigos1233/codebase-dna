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
| Writing code against a fingerprint | `SKILL.md` + your project's `dna.md` |
| Auditing a codebase | `SKILL.md` + `vocabulary/` + `audit/` + `template/dna.template.md` |
| Filling a fingerprint manually | `SKILL.md` + `vocabulary/` + `template/` |

For system prompt injection (e.g. API usage), include the content of the relevant files directly. Use `template/dna.minimal.template.md` for the fingerprint itself — it's designed to fit comfortably in a system prompt.

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

Add to your project's `CLAUDE.md` (or equivalent):

```markdown
@dna.md
```

Or reference it explicitly:

```
Use the project fingerprint in dna.md to guide your code style.
```

The agent will load the fingerprint and use it to match existing patterns when writing new code. If it encounters an unfamiliar term, it will load only the relevant vocabulary section — not the full skill.

### Fill a fingerprint manually

Copy `template/dna.template.md` (full) or `template/dna.minimal.template.md` (for system prompts) to your project root and fill it in. The `vocabulary/` files define every axis and property precisely if you need a reference.

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
└── template/
    ├── dna.template.md         # Full fingerprint template
    └── dna.minimal.template.md # Stripped version for system prompt injection
```
