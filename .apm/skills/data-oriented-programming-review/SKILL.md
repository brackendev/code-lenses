---
name: data-oriented-programming-review
description: >-
  Review code against Yehonathan Sharvit's Data-Oriented Programming
  principles: code separated from data, data represented with generic data
  structures, immutable data, and data schemas kept separate from the data.
  Use when asked for a "data-oriented programming review" or "DOP review",
  or when the user asks whether code mixes data and behavior, mutates shared
  data, or locks data into entity classes. Read-only report.
argument-hint: "[scope or options...]"
user-invocable: true
---

# Data-Oriented Programming Review

Review code against the four principles of Data-Oriented Programming by Yehonathan Sharvit (*Data-Oriented Programming*, Manning, 2022), summarized in [Principles of Data-Oriented Programming](https://blog.klipse.tech/dop/2022/06/22/principles-of-dop.html). The `data-oriented-programming` skill, next to this skill's directory, holds this package's short summary of each principle and the links to Sharvit's excerpts. Read `data-oriented-programming/SKILL.md` before reviewing. Apply the principles to the reviewed code's stack following `lenses-fix/references/stack-adaptation.md`.

All review output uses direct, professional voice. Cite each principle by number (`DOP #3`).

## Arguments

Interpret naturally. This skill is a pure report. It does not mutate the workspace and carries no `--report` flag (there is nothing to invert).

| Input | Action |
|-------|--------|
| (no argument) | Review changed files only (staged + unstaged) |
| `all` | Review full codebase (sample high-risk and high-traffic modules) |
| `<path>` `<glob>` | Review files under the path or matching the pattern |

Optional modifiers can appear anywhere in user input:

| Intent | Examples |
|--------|----------|
| Focus | `data model`, `state`, `schemas`, `entities` |
| Output shape | `verdict only`, `no approvals`, `problems only` |

Output-shape modifiers change what the report shows, not how much of the scope the review covers.

## Context Gathering

Run these commands for scope discovery:

- `git status --short`
- `git diff --name-only`
- `git diff --cached --name-only`

For explicit scope args, collect files from provided paths.
Filter to source/test/config files relevant to behavior. Skip generated files, lock files, vendored dependencies, and binaries.

If no files are found (and no explicit scope), ask user what to review.

## Review Process

### 1) Build Evidence Per File

For each file in scope:

1. Read file (or diff hunk when scoped to changes).
2. Identify how entities are represented: specific classes, records, generic maps, or arrays.
3. Find where data changes: in-place mutation, new versions, or references swapped to new values.
4. Find where data shape is checked: class definitions, schemas, validation functions, or nowhere.
5. Note which functions read data held in their own context (fields, closures) instead of receiving it.

### 2) Detect Departures From the Principles

The severity tiers and their names are this package's review scale for ranking findings, not part of Sharvit's book. Cite the principle in each finding.

#### ENTANGLED (High)

Data and behavior bound together, or data changed where other code can see it:

- Objects that hold data and the methods that change it, shared across modules (DOP #1)
- Functions that close over mutable state instead of receiving data as arguments (DOP #1)
- Shared data mutated in place while other code holds a reference to it (DOP #3)

#### CONFINED (Medium)

Data locked into shapes that only specific code can use:

- A class per entity where a generic structure would let existing collection functions apply, and the class adds no behavior or guarantee the code relies on (DOP #2)
- New classes created only to add or remove a field (DOP #2)
- Defensive copies made because the data is mutable (DOP #3)

#### IMPLICIT (Low)

Shape knowledge that is missing or scattered:

- Data that crosses a boundary with no schema and no check (DOP #4)
- The same shape checked by hand in several places instead of one schema (DOP #4)

### 3) Identify Data-Oriented Patterns

Call out code that follows the principles well, citing the number: for example, static or standalone functions over data passed in (DOP #1), maps and lists processed with generic collection functions (DOP #2), new versions returned instead of mutation (DOP #3), and one schema validated at the boundary (DOP #4).

### 4) Suggest Changes

For each departure, propose the alternative the cited principle describes, adapted to the stack. State the cost Sharvit names for that principle when it applies to the code at hand. Suggest one entity or one function at a time, not a rewrite.

## False-Positive Guardrails

- Do not flag classes that a framework requires (ORM entities, serialization contracts, UI component classes). Report them as departures with the framework constraint as the reason.
- Do not flag a specific type where it carries a guarantee the code relies on, such as a parsed domain type from Parse Don't Validate. Report the representation tension instead of a violation.
- Do not flag local, short-lived mutation that no other code can observe, such as a builder inside one function.
- Do not assume missing context or claim certainty without evidence from code, tests, or explicit constraints. Mark uncertain findings as uncertain.
- Prefer fewer high-confidence findings over many weak findings.
- Every finding must include specific file reference (`file:line`) and concrete symptom.
- If uncertain why code exists, say so and reference Chesterton's Fence.

## Output Contract

Use this structure unless user asked for a shorter variant:

```markdown
## Data-Oriented Programming Review: [scope]

### Departures

**[file:line]** -- [ENTANGLED|CONFINED|IMPLICIT] DOP #N: [principle]
[What the code does]
[What the principle asks for, and the cost Sharvit names when it applies]
[Change: alternative adapted to the stack, with a code example if helpful]

### Data-Oriented Patterns Found

**[file:line]** -- DOP #N: [what follows the principle and why]

### Verdict

- **Data orientation:** Data-oriented / Mostly data-oriented / Mixed / Object-bound
- **Principle coverage:** [which principles are relevant and how well they are followed]
- **Framework departures:** [code that departs because of a framework or project constraint, with the constraint, or "none"]
- **One-sentence summary:** [single sentence]

### Suggested Changes

1. [highest-impact change, citing the principle number]
2. [next change]
3. [next change]

| Metric | Count |
|--------|-------|
| Files reviewed | N |
| ENTANGLED | N |
| CONFINED | N |
| IMPLICIT | N |
| Data-oriented patterns | N |
| Data orientation | [level] |
```

If no issues found, say so explicitly and still provide the metric table.
