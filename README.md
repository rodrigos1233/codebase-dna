# codebase-dna

A Claude Code skill that gives agents a shared vocabulary for understanding
how a codebase is written — not what it does.

The fingerprint (`dna.md`) captures only what an agent cannot see by reading
a single file: structural constraints, state contracts, established idioms,
known traps, and areas that must stay simple. It is designed to be loaded
at agent startup, not read by humans.

---

## How it works

1. Install the skill once (shared across projects)
2. Audit each codebase to produce a `dna.md` fingerprint
3. Reference both from the project's `CLAUDE.md` so agents load them automatically

Without step 3, the skill and fingerprint exist but are never used.

---

## Installation

**Claude Code** — clone into the skills directory:

```bash
git clone https://github.com/rodrigos1233/codebase-dna ~/.claude/skills/codebase-dna
```

**Codex** — clone and symlink into the agents skills directory:

```bash
git clone https://github.com/rodrigos1233/codebase-dna ~/.codex/codebase-dna
mkdir -p ~/.agents/skills
ln -s ~/.codex/codebase-dna ~/.agents/skills/codebase-dna
```

**Any other agent** — copy or reference `SKILL.md` and the relevant subdirectories directly. The skill is plain markdown; no tooling required.

---

## Auditing a codebase

From within the project directory, start a Claude Code session and run:

```
Load the codebase-dna skill from ~/.claude/skills/codebase-dna/SKILL.md
and follow audit/prompt.md to produce a dna.md for this project.
```

The agent will read the codebase, apply admission rules, and write a
`dna.md` at the project root.

Review the output before committing — mark any entries as aspirational
(`current:` / `target:`) where the fingerprint reflects intent rather than
current reality.

---

## Connecting it to your project (required)

The fingerprint and skill are only useful if agents load them. Add the
following to your project's `CLAUDE.md`:

```markdown
## Codebase fingerprint

This project has a codebase fingerprint at `dna.md`. Before writing any
code, load it:

- For a focused task: load `~/.claude/skills/codebase-dna/SKILL.md`,
  then run `scope/prompt.md` with this task description and `dna.md`
  to generate a compact task-scoped injection. Use that instead of the
  full fingerprint.
- For broad or cross-cutting work: load `dna.md` directly.
- If a term in `dna.md` is unclear: load
  `~/.claude/skills/codebase-dna/vocabulary/sections.md` for that
  section's definition only.
```

This is a manual step. It must be done once per project after the initial
audit. For Codex, add the equivalent block to your project's `AGENTS.md`.
The content is the same — only the file name differs.

---

## Keeping the fingerprint current

A fingerprint drifts as the codebase evolves. Re-audit when:

- A significant refactor changes established patterns
- A new architectural boundary is introduced
- A known-debt entry is resolved
- An agent reports a contradiction between `dna.md` and what it observes

To re-audit, run the same audit prompt again. Diff the output against the
existing `dna.md` and update only what has changed.

---

## File map

```
SKILL.md                    entry point and navigation

vocabulary/
  sections.md               definitions and admission rules for all seven sections
  meta.md                   evidence levels, aspirational markers, drift

audit/
  sampling-strategy.md      what to read and why
  prompt.md                 complete audit procedure

scope/
  prompt.md                 filters a filled dna.md to a 20–40 line task-scoped injection

template/
  dna.template.md           full fingerprint template
  dna.minimal.template.md   compact template for manual filling
  dna.inject.md             task-scoped injection format (output of scope/prompt.md)
```

---

## Sections in a filled dna.md

| Section | What it captures |
|---|---|
| `stack` | Libraries committed to per domain — use these, not alternatives |
| `boundaries` | Structural constraints; auth placement; process rules |
| `state-contracts` | Where state lives; what must be updated together |
| `patterns` | Idioms not inferrable from one file |
| `dead-ends` | Approaches tried and abandoned — do not revisit |
| `known-debt` | Patterns present but not to be imitated — location-scoped |
| `simplicity-zones` | Areas that must not be abstracted or improved |

---

## Design principles

- The fingerprint captures only what is not locally legible. An agent
  reading one file already sees type strictness, naming conventions, and
  error handling style. Those do not belong here.
- Every entry carries an `evidence:` level. `inferred` entries are strong
  hints, not hard rules.
- Admission rules exist to keep sections honest. A short fingerprint with
  high-confidence entries is better than a long one with folklore.
- Running two agents on the same audit and diffing the outputs is a useful
  quality check. Disagreement between audits is signal, not noise.
