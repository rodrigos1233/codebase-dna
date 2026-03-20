---
name: codebase-dna
description: Use when auditing a codebase to produce a dna.md, writing code against an existing fingerprint, scoping a fingerprint to a specific task, or filling one manually.
---

# codebase-dna

`dna.md` is the project fingerprint: a compact description of how the codebase is written, not what it does. It captures only what an agent cannot see by reading a single file.

## Modes

**Writing code against an existing fingerprint**
- Focused task: use `scope/prompt.md` with the project's `dna.md` to generate a `dna.inject.md`, then use that instead of the full fingerprint.
- Broad or cross-cutting work: load the project's full `dna.md` directly.

**Auditing a codebase to produce a fingerprint**
Load all of the following, in this order:
1. `vocabulary/sections.md` — section definitions and admission rules
2. `vocabulary/meta.md` — evidence levels, aspirational markers, drift
3. `audit/sampling-strategy.md` — what to read and in what order
4. `audit/prompt.md` — the audit procedure
5. `template/dna.template.md` — the output format

**Filling a fingerprint manually (without a full audit)**
Load `vocabulary/sections.md`, `vocabulary/meta.md`, and `template/dna.template.md` or `template/dna.minimal.template.md`.

## Vocabulary loading rule

If a term or entry in an existing `dna.md` is unclear, load `vocabulary/sections.md` and read only the definition for that section. Do not preload the full vocabulary speculatively.

## File map

```
vocabulary/
  sections.md          — definitions and admission rules for all seven sections
  meta.md              — evidence levels, aspirational markers, drift

template/
  dna.template.md      — full fingerprint template with comments and all fields
  dna.minimal.template.md  — compact template, no comments, evidence omitted
  dna.inject.md        — task-scoped injection format (output of scope/prompt.md)

audit/
  sampling-strategy.md — what to read and why; large-codebase constraints
  prompt.md            — complete audit procedure

scope/
  prompt.md            — filters a filled dna.md to a 20–40 line task-scoped injection
```

## Sections in a filled dna.md

| Section | What it captures |
|---|---|
| `stack` | Libraries committed to per domain — use these, not alternatives |
| `boundaries` | Structural constraints on what can call what; auth placement; enforcement mechanism |
| `state-contracts` | Where state lives; which mechanism to use when; what must be updated together (change-coupling) |
| `patterns` | Established idioms not inferrable from one file |
| `dead-ends` | Approaches tried and abandoned — do not revisit |
| `known-debt` | Patterns present but not to be imitated — location-scoped |
| `simplicity-zones` | Areas that must not be abstracted or improved |

Do not load vocabulary or audit docs unless the active mode requires them.
