# Audit Prompt

This is a ready-to-use prompt for an agent tasked with auditing a codebase and producing a draft `dna.md`. Copy this prompt directly or adapt it for your orchestration system.

---

## Prompt

You are auditing a codebase to produce a structured fingerprint called `dna.md`. The fingerprint describes *how* the code is written, not what it does. It will be used by future agents to write new code consistently without reading the whole codebase.

### Step 1: Load your reference materials

Before reading any code, load these files from the `codebase-dna` skill:

1. `vocabulary/sliders.md` — definitions for all 18 slider axes
2. `vocabulary/binaries.md` — definitions for all 6 binary properties
3. `vocabulary/meta.md` — how to use descriptive vs aspirational positions, dead ends, simplicity zones, and drift
4. `audit/sampling-strategy.md` — which files to read and in what order

Read all four files completely before proceeding. The vocabulary must be loaded before you read any code, or your classifications will be inconsistent.

### Step 2: Read the codebase

Follow the reading order in `audit/sampling-strategy.md`:

1. Project root files (README, package manifest, config files, existing agent docs)
2. Entry points and middleware
3. One full feature slice (route → handler → service → data layer)
4. Test files for that slice
5. Config and infrastructure files

As you read, take notes per axis. Do not assign positions yet — just record what you observe.

### Step 3: Classify each axis

For each axis in `vocabulary/sliders.md` and each property in `vocabulary/binaries.md`:

1. State what you observed in the code that informs this axis.
2. Assign a position on the 0–10 scale (sliders) or true/false (binaries).
3. Ask: Does the observed code reflect current reality, or does there appear to be an intended direction that the code is moving toward? If the latter, record an aspirational position.
4. Write notes that would help a future agent understand the reasoning — especially for positions near the extremes or where you found inconsistency.

Do not guess. If you did not read enough code to classify an axis, say so in the notes field rather than fabricating a position.

### Step 4: Identify dead ends, simplicity zones, and known debt

**Dead ends:** Did you find patterns that appeared to be in the process of being replaced, or that the team documented as abandoned? Record them with why they were abandoned.

**Simplicity zones:** Are there areas of the codebase that are intentionally simple and clearly should stay that way? Record them with the rule.

**Known debt:** Are there patterns in the codebase that clearly violate the team's own standards — present for historical reasons but not to be imitated? Record them here, not as the descriptive position.

### Step 5: Record preferred libraries

For each problem domain the codebase addresses, record which library or approach is used. Use the categories in the `preferred-libs:` section of `template/dna.template.md`. Add categories as needed for the specific project.

### Step 6: Flag contradictions and inconsistencies

Before writing the final output, explicitly list:

- Any axis where different parts of the codebase gave contradictory signals
- Any case where the README, CLAUDE.md, or existing docs stated something different from what the code showed
- Any axis where you had too little evidence to be confident

Include these as notes in the relevant axis sections and as a summary at the top of the fingerprint under `audit-notes:`.

### Step 7: Write the output

Fill `template/dna.template.md` with your findings. Rules:

- Every axis must have a `position:` value. Do not leave axes blank.
- If you have no evidence for an axis, set `position: unknown` and explain in `notes:`.
- Use `aspirational:` only when there is clear evidence of intended direction different from current state.
- `notes:` must be present for any position at 0–2 or 8–10, and for any axis with inconsistency.
- The `dead-ends:`, `simplicity-zones:`, and `known-debt:` sections must be present even if empty.
- Do not invent information. The fingerprint will be used to generate code. False certainty is worse than recorded uncertainty.

Output the filled `dna.md` as a single markdown document. Do not summarize or explain it — just output the document.
