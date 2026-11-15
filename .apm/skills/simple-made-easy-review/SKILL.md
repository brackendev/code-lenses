---
name: simple-made-easy-review
description: >-
  Review code for entangled concerns using the design ideas of Rich Hickey's
  talk Simple Made Easy: state that ties a value to time, classes around
  plain information, inheritance and repeated branching, collaborators fixed
  inside a unit, direct calls that fix when and where work happens, and
  constructs chosen because they are familiar. Use when asked for a "Simple
  Made Easy review" or a "complecting review", or when the user asks whether
  code is complected, entangled, or simple versus easy. Read-only report.
argument-hint: "[scope or options...]"
user-invocable: true
---

# Simple Made Easy Review

Review code against the design ideas of Rich Hickey's talk [Simple Made Easy](https://www.infoq.com/presentations/Simple-Made-Easy/) (Strange Loop, 2011). The `simple-made-easy` skill, next to this skill's directory, holds this package's numbered summaries of the talk. Read `simple-made-easy/SKILL.md` before reviewing. Apply the principles to the reviewed code's stack following `lenses-fix/references/stack-adaptation.md`.

All review output uses direct, professional voice. Cite each principle by number (`SME #3`). Every finding names the two concerns that the code ties together.

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
| Focus | `state`, `dispatch`, `interfaces`, `component connections` |
| Output shape | `verdict only`, `no approvals`, `problems only` |

Output-shape modifiers change what the report shows, not how much of the scope the review covers.

## Context Gathering

With no scope argument, run these commands for scope discovery:

- `git status --short`
- `git diff --name-only`
- `git diff --cached --name-only`

For explicit scope args, collect files from provided paths and skip the commands above.
Filter to source/test/config files relevant to behavior. Skip generated files, lock files, vendored dependencies, and binaries.

When a path holds more files than the review can read in full, read the largest files and the ones the others depend on, sample the rest, and report how many files were read out of how many are in scope.

If no files are found (and no explicit scope), ask user what to review.

## Review Process

### 1) Build Evidence Per File

For each file in scope:

1. Read file (or diff hunk when scoped to changes).
2. List the state the code holds or reads, and where references to that state travel.
3. Note how variants are dispatched (branches, inheritance, interfaces, or tables) and whether the same branch appears in more than one place.
4. Note how each unit gets its collaborators, and which calls between components are direct.
5. Note where rules and policy live, and how information is represented.
6. For each candidate finding, name the two concerns that are tied together, and name what exposes the tie. For two pieces of code, that is a realistic change to one that would force a change to the other. For state, where the tie is between a value and the moment it is read, that is a realistic event at run time, such as a reload or a concurrent update, that makes two reads disagree. Discard the candidate when neither can be named.

### 2) Detect Entanglement

The severity tiers and their names are this package's review scale for ranking findings, not part of the talk. Cite the principle in each finding.

#### BRAIDED (High)

Concerns interleaved so that one cannot be changed, tested, or understood without the other, with an effect that reaches beyond one module:

- Shared mutable state read at several points of a computation that spans modules, or a reference to it passed through layers that compute from a single read and never read later or change it (SME #3)
- A module that relies on an unstated detail of another, such as an assumed call order, a value it assumes is never returned, or a shared mutable structure, behind a boundary that looks clean (SME #2)
- A unit that decides a rule, chooses the concrete collaborators that carry it out, and performs the effect, all in one place (SME #9)
- Objects that bundle information with the state changes on it and are passed between modules (SME #3, SME #4)

#### COUPLED (Medium)

A construct that ties two concerns where the stack offers a way to keep them apart:

- State read more than once inside one unit where one read would serve, so that the unit can combine values from two moments (SME #3)
- The same switch, match, or type test on the same variants repeated across functions (SME #6)
- Inheritance used to share code or to vary behavior (SME #6)
- A wide interface whose implementations leave members empty, or whose callers each use a small part of it (SME #9)
- Collaborators constructed or looked up inside a unit instead of received as arguments (SME #9)
- A direct call between components that should not know each other, or that run at different rates or times (SME #9)
- A class for each kind of information that adds no behavior and no guarantee the code relies on (SME #4)
- Business rules written as conditionals spread across handlers (SME #8)

#### CONVENIENT (Low)

A construct that is familiar where a simpler one is at hand, with a local effect:

- A hand-written loop that interleaves what is computed with how the iteration proceeds, where collection operations exist (SME #8)
- A custom text format, a command built by joining strings, or special syntax where plain data would serve (SME #7)
- Methods on a data holder that use no instance state (SME #5)

SME #1 has no detection item of its own, because code rarely records why a construct was chosen. Cite it next to another principle only when a comment, a commit message, or a design note in the project gives familiarity or speed of setup as the reason.

### 3) Identify Simple Patterns

Call out code that follows the principles well, citing the number: for example, a function that takes a value where a reference to state was available (SME #3), information kept as plain data and processed with generic collection functions (SME #4), a small set of operations that new variants extend without edits elsewhere (SME #6), a description held as data and interpreted by one function (SME #7), and collaborators received as arguments (SME #9).

### 4) Suggest Changes

For each finding, propose the alternative the cited principle describes, adapted to the stack. State the cost of the alternative when it applies to the code at hand: more parts, more indirection, a performance cost, or a construct the team does not know. Suggest one separation at a time, not a rewrite.

The Suggested Changes section of the report ranks the changes by their effect on the code, not by tier, and lists at most five. Each entry names its finding and does not repeat the finding's text.

## False-Positive Guardrails

- Do not report unfamiliar, long, or verbose code as complex, and do not accept familiar code as simple. A finding exists only when two named concerns are tied together and could change independently.
- Do not recommend an interface, a queue, or an extension point for a variation that does not exist in the code or in a stated requirement. More parts are justified only by concerns that are entangled today. Asking a caller to stop waiting for a callee adds no part when the component already has a way to run work without waiting, such as an executor, a scheduler, or a task runner. Recommend that mechanism before a new queue.
- Do not flag a branch on a closed set of variants that is written in one function. The finding is the same branch on the same variants written in more than one function.
- Do not flag a direct call for sequential work inside one component.
- Do not flag a reference to state that is passed to a unit which must read the state later than its caller, or must change it. The finding is a reference passed to code that computes from a single read.
- Do not give a tier to a construct that a framework or an established project convention requires (ORM entities, class-based components, a required base class, a handler signature the project fixes). Report it under Departures with the constraint as the reason, and count it in the Departures row, not in a tier row. A convention that the reviewed scope defines for itself is not a constraint, and code that follows it can be a finding.
- Do not flag local, short-lived mutation that no other code can observe, such as a builder inside one function.
- Do not recommend a mechanism the language lacks. Recommend the closest mechanism the stack has.
- Do not assume missing context or claim certainty without evidence from code, tests, or explicit constraints. Mark uncertain findings as uncertain.
- Prefer fewer high-confidence findings over many weak findings.
- Every finding must include specific file reference (`file:line`) and concrete symptom.
- If uncertain why code exists, say so and reference Chesterton's Fence.

## Output Contract

Use this structure unless user asked for a shorter variant:

```markdown
## Simple Made Easy Review: [scope]

**Stack:** [language, paradigm, typing, and data representation of each component, with file references]
**Files read:** [N of M in scope, and how the others were sampled, or "all"]

### Entanglements

**[file:line]** -- [BRAIDED|COUPLED|CONVENIENT] SME #N: [principle]
[The two concerns the code ties together]
[A change to one that forces a change to the other, or a run-time event that makes two reads disagree]
[Change: alternative adapted to the stack, with its cost and a code example if helpful]

### Simple Patterns Found

**[file:line]** -- SME #N: [what follows the principle and why]

### Verdict

- **Simplicity:** Simple / Mostly simple / Mixed / Entangled
- **Principle coverage:** [which principles are relevant and how well they are followed]
- **Departures:** [code that departs because a framework or an established project convention requires it, with the constraint, or "none"]
- **One-sentence summary:** [single sentence]

### Suggested Changes

1. [the change with the largest effect, naming its finding and principle number]
2. [next change, up to five in total]

| Metric | Count |
|--------|-------|
| Files read | N of M |
| BRAIDED | N |
| COUPLED | N |
| CONVENIENT | N |
| Departures | N |
| Simple patterns | N |
| Simplicity | [level] |
```

Choose the simplicity level by this scale, which is this package's own:

- **Simple:** no BRAIDED and no COUPLED findings.
- **Mostly simple:** no BRAIDED findings, and no COUPLED finding that recurs across several units.
- **Mixed:** one BRAIDED finding, or a COUPLED finding that recurs across several units.
- **Entangled:** several BRAIDED findings, or one that the main flow of the scope depends on.

If no issues found, say so explicitly and still provide the metric table.
