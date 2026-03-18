---
name: codebase-dna
description: Audit a codebase, use an existing `dna.md`, or generate/fill a fingerprint with the right load path.
---

# codebase-dna

`dna.md` is the project fingerprint: a compact description of how the code is written, not what it does.

## Modes

- Focused implementation task: use `scope/prompt.md` with the project's `dna.md` to generate a fresh `dna.inject.md`, then use that instead of the full fingerprint.
- Broad or cross-cutting work: load the project's full `dna.md`.
- Audit or manual fingerprinting: load `vocabulary/sliders.md`, `vocabulary/binaries.md`, `vocabulary/meta.md`, and `template/dna.template.md` or `template/dna.minimal.template.md`; add `audit/sampling-strategy.md` and `audit/prompt.md` only when auditing code.

Do not load vocabulary or audit docs unless the active mode requires them. Do not preload extra files.
