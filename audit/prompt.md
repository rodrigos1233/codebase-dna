# Audit Prompt

This is a ready-to-use prompt for an agent tasked with auditing a codebase and producing a `dna.md`. Copy it directly or adapt it for your orchestration system.

---

## Prompt

Your job is not to classify this codebase. Your job is to find rules a local read would miss.

An agent working on a single file already sees naming conventions, type annotations, error handling style, and which libraries are imported. The fingerprint you are producing captures only what that agent cannot see: structural constraints, established conventions that required deliberate choices, and traps left by past decisions.

If an entry you are considering could be inferred by reading the file being modified, leave it out.

### Step 1: Load your reference materials

Before reading any code, load from the `codebase-dna` skill:

1. `vocabulary/sections.md` — definitions and admission rules for all seven sections
2. `vocabulary/meta.md` — evidence levels, aspirational markers, and drift
3. `audit/sampling-strategy.md` — which files to read and in what order

Read all three completely before reading any code. The admission rules must be loaded before you read anything, or you will record entries that do not belong.

### Step 2: Read the codebase

Follow `audit/sampling-strategy.md` in order:

1. Existing agent documentation (CLAUDE.md, AGENTS.md, decisions logs)
2. Configuration and enforcement files (package manifests, lint config, tsconfig)
3. Entry points and module boundary definitions (main file, router, middleware stack, module index files)
4. One full feature slice — read for boundary and state signals, not style classification
5. Git history — dead end evidence only
6. Test files — boundary and state signals only

As you read, take notes on potential entries. Do not commit to a section yet — just record what you observe and where.

### Step 3: Apply admission rules to your notes

For each potential entry, apply the section's admission rules explicitly. Do this before writing output, not during.

**stack:** Would choosing the wrong library create visible inconsistency? If not, omit.

**boundaries:** Does violating this rule produce a structural or security defect? Is the enforcement mechanism identifiable? If not, omit.

**state-contracts:** Does violating this cause a runtime defect or hard-to-trace bug? Is this invisible from the file being modified? If not, omit.

**patterns — two gates, applied as rejection tests:**
1. Apply as a cut: if you are writing `why-non-obvious:` and completing it feels like a stretch — if you find yourself explaining what the code does rather than why an agent would miss it — the entry is cut.
2. Apply as a cut: name the concrete wrong action an agent would take without this entry. If you cannot name a specific wrong action, the entry is cut. "The agent might not know about this" is not a wrong action.

**dead-ends:** Do you have evidence — documentation, commit history, a clear replacement in active code, or repeated absence in new code? "This looks old" is not sufficient. If you cannot cite evidence, do not record it.

**known-debt:** Does the entry name a specific file, directory, or module? Does it say what to do instead? If either is missing, do not record it.

**simplicity-zones:** Is this a deliberate local anti-abstraction decision, not a general preference? Is it specific enough to be actionable? If it is describing general YAGNI sentiment, omit it.

Cut any entry that does not pass its admission rule. It is better to have a short fingerprint that is trustworthy than a long one that includes folklore and inference passed off as fact.

### Step 4: Assign evidence levels

For every entry that survives Step 3, assign an evidence level:

- `observed` — you saw this pattern directly in active code
- `documented` — it is stated in CLAUDE.md, README, a decisions log, commit message, or comment
- `inferred` — you reasoned from partial evidence; you did not directly observe or read a statement

Do not assign `observed` if you only saw it in one file. Do not assign `documented` if you are paraphrasing what you read — only if the source explicitly states the rule. When in doubt, use `inferred`.

### Step 5: Identify aspirational entries

For `known-debt` and `patterns` entries only: is there an active migration or stated direction that changes what new code should do? If so, add `current:` and `target:` fields. If the distinction does not change what an agent would write, omit the aspirational fields.

### Step 6: Flag contradictions and gaps

Before writing output, list explicitly:

- Any area where different parts of the codebase gave contradictory signals (e.g., some modules follow a boundary rule, others don't)
- Any area where documented conventions contradicted what the code shows
- Any section where you had too little evidence to be confident

Record these in `audit-notes`. They are not failures — they are information.

### Step 7: Write the output

Fill `template/dna.template.md`. Rules:

- Every list section may be empty (`[]`) if there is nothing to record. Do not fabricate entries to fill a section.
- `dead-ends` may be empty. An empty `dead-ends` is honest. A fabricated dead end is harmful.
- `evidence:` is required on every entry. An entry without an evidence level is incomplete.
- `audit-notes` must contain anything from Step 6, even if brief.
- Do not invent information. False certainty in a fingerprint causes agents to make wrong decisions confidently.

Output the filled `dna.md` as a single markdown document. Do not summarize or explain it outside the document.

---

## Two-agent validation

Running a second agent with access to files not covered in the first pass is a useful quality check. Structured disagreement between two audits is signal — entries present in one but not the other should be reviewed against admission gates, not automatically merged or discarded. An entry in the second audit that the first missed is worth keeping only if it passes admission; an entry in the first that the second dropped may indicate the first over-captured.
