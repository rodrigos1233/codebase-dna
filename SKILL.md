---
name: codebase-dna
description: Use when auditing a codebase to produce a structured fingerprint, writing code consistently against an existing fingerprint, or filling/correcting a dna.md file
---

# codebase-dna

A shared vocabulary and tooling for describing, auditing, and using codebase style fingerprints.

## What is a `dna.md`?

A `dna.md` is a structured fingerprint of a codebase — its style, constraints, preferred patterns, and known debt. It is not a summary of what the code does. It describes *how* the code is written, so an agent can produce new code that fits without reading the whole codebase.

A `dna.md` can be:
- Stored at the project root and referenced from `CLAUDE.md`
- Injected directly into a subagent system prompt
- Filled by a human, an audit agent, or both

## What to Load

**Writing a feature or fix against an existing fingerprint:**
- Load the project's `dna.md` only. Nothing else from this skill is needed.

**Auditing a codebase to produce a draft `dna.md`:**
- Load `vocabulary/sliders.md`
- Load `vocabulary/binaries.md`
- Load `vocabulary/meta.md`
- Load `audit/sampling-strategy.md`
- Load `audit/prompt.md`
- Use `template/dna.template.md` as the output structure

**Filling or correcting a fingerprint manually:**
- Load `vocabulary/sliders.md`
- Load `vocabulary/binaries.md`
- Load `vocabulary/meta.md`
- Use `template/dna.template.md` or `template/dna.minimal.template.md`

## Vocabulary Overview

- `vocabulary/sliders.md` — 18 axes with two extremes each. Every axis is a spectrum, not a right answer.
- `vocabulary/binaries.md` — 6 binary (yes/no) properties with verification instructions.
- `vocabulary/meta.md` — Cross-cutting concepts: descriptive vs aspirational positions, dead ends, simplicity zones, and drift detection.

## Design Principles

- The vocabulary defines spectrums. It does not prescribe correct positions.
- Positions are descriptive by default. Aspirational positions must be explicitly marked.
- Two agents reading the same codebase should produce the same fingerprint.
