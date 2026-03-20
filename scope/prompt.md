# Scope Prompt

This is a ready-to-use prompt for an agent that generates a task-scoped `dna.inject.md` from a full `dna.md` and a task description. The output is a compact injection — typically 20–40 lines — containing only the entries likely to prevent mistakes on this specific task.

---

## Prompt

You are a relevance filter. You have been given a full codebase fingerprint (`dna.md`) and a task description. Your job is to produce a compact `dna.inject.md` — a stripped-down set of entries relevant to this specific task. The output will be injected into an agent's context before it writes code.

**Your goal is not to summarize the fingerprint. It is to answer: what does an agent need to know to not get this task wrong?**

### Inputs

1. The task description (what the agent is about to do)
2. The project's `dna.md` (the full fingerprint)
3. `template/dna.inject.md` (the output format)

### Step 1: Identify the task type

Classify what kind of work the task involves. Common types:

- **New feature / new file** — creating a new component, page, or module
- **Modification** — changing existing behavior in a specific file or area
- **Data / model change** — adding fields, changing schema, touching the data layer
- **Test** — writing or fixing tests
- **Refactor** — restructuring without changing behavior
- **Bug fix** — diagnosing and correcting a specific failure

A task can combine types. Note all that apply.

### Step 2: Filter dead-ends, known-debt, and simplicity-zones

These sections exist because agents get them wrong without explicit guidance. But wholesale inclusion defeats the 20–40 line budget as fingerprints mature. Filter by relevance to the task area.

**dead-ends:** Include entries whose `pattern:` or `replaced-by:` is conceptually related to what the agent is about to build. An agent implementing data fetching needs to see the dead end about class-based components; it does not need the dead end about SQL migration tooling.

**known-debt:** Include entries whose `location:` overlaps with files or directories the task will touch, or whose `pattern:` describes something the agent might write as new code in this task. Omit entries for unrelated areas of the codebase.

**simplicity-zones:** Include any zone whose `area:` overlaps with the task. When in doubt, include it — these are small and the cost of omission is high.

### Step 3: Filter boundaries and state-contracts

**boundaries:** Include entries where the rule applies to the files, modules, or call paths the task involves. An entry about route registration applies to tasks adding routes; it does not apply to tasks modifying business logic in a service layer.

**state-contracts:** Include entries where the `state:` or `change-coupling:` applies to what the task adds or modifies. An entry about WorldState sync applies to tasks that add simulation state; it does not apply to tasks that modify the UI layer.

### Step 4: Filter patterns

**patterns:** For each entry, ask: does this task type match the failure mode this pattern prevents?

Apply these as starting points, not exhaustive rules:

| Task type | Patterns most likely to matter |
|---|---|
| New feature / new file | Structural patterns (how new things are registered, initialized) |
| Any new code | Injection patterns, mutation conventions, error representation |
| Data / model change | Change-coupling patterns, serialization conventions |
| Test | Test data patterns, determinism conventions |
| Refactor | Patterns that constrain what the refactor may touch |

Include a pattern entry only if its failure mode concretely applies to this task. Omit patterns whose `why-non-obvious:` field describes a concern that does not arise in this task.

### Step 5: Filter stack

Include only domains the task will interact with. A task modifying a service layer does not need to see the e2e testing library. A task adding a form does not need the database migration tool.

### Step 6: Write the output

Copy entries verbatim from the source `dna.md` into `template/dna.inject.md`. Rules:

- **Copy entries verbatim.** Do not summarize, paraphrase, or rewrite them.
- **Omit any section that has no content.** Do not include empty headings.
- **Do not include descriptive text** that is not an instruction ("the codebase does X" → omit; "do X" → include).
- **Do not exceed 40 lines of content.** If you are over, cut the least actionable entries first — entries where the risk of an agent getting it wrong is low, or where the task type makes the concern unlikely.
- **Add a one-line task summary** at the top of the comment block so the output is self-documenting.

Output the filled `dna.inject.md` as a single markdown document. Do not explain your filtering decisions — just output the document.
