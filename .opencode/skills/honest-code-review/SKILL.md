---
name: honest-code-review
description: Review code for dishonest patterns using the Honest Code constructs (11 from honestcode.software, 1 extended)
argument-hint: "[scope or options...]"
user-invocable: true
disable-model-invocation: true
---

# Honest Code Review

Review code against the [Honest Code](https://honestcode.software) constructs. Constructs 1 through 11 are by Adam Zachary Wasserman. Construct 12 extends the philosophy with Gary Bernhardt's Functional Core, Imperative Shell pattern. Identify dishonest patterns (crime scenes) and suggest rescues (honest alternatives).

All review output uses direct, professional voice. Reference constructs by name and number.

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
| Focus | `tests only`, `architecture`, `data layer`, `state management` |
| Output shape | `verdict only`, `no approvals`, `problems only` |

Always run the full review flow. Do not provide reduced-depth modes.

## Context Gathering

Run these commands for scope discovery:

- `git status --short`
- `git diff --name-only`
- `git diff --cached --name-only`

For explicit scope args, collect files from provided paths.
Filter to source/test/config files relevant to behavior. Skip generated files, lock files, vendored dependencies, and binaries.

If no files are found (and no explicit scope), ask user what to review.

## The Constructs (Cheat Sheet)

Use this framework to evaluate code. Each construct defines an "instead of" (dishonest) and "use" (honest) pattern.

| # | Construct | Instead Of (Dishonest) | Use (Honest) |
|---|-----------|----------------------|--------------|
| 1 | Data Is Data | Class with fields + methods | TypedDict, record, struct, plain object |
| 2 | Input In, Output Out | Methods mutating self, class hierarchy polymorphism | Pure functions, dict-lookup dispatch |
| 3 | One Source of Truth | Client + server state sync | DOM or database as single source |
| 4 | Declare, Don't Instruct | Imperative DOM, event listener wiring | HTML attributes, browser-native APIs |
| 5 | Compose Flat, Never Deep | Inheritance, handler hierarchies, middleware stacks | pipe/compose, independent functions |
| 6 | Let It Crash | Swallowed errors, nested try-catch, inline retry | Raise at source, handle at boundary, supervision |
| 7 | Profile First, Fix Architecture | Cache without measurement, multiple service calls | Single query with indexes, measure first |
| 8 | Boring Tests | Mock-heavy setup, wrapper components for testing | assert f(input) == expected, HTTP + check HTML |
| 9 | Constrain AI | Accept 500-line AI-generated classes | Honest architecture as prompt, small functions |
| 10 | Declare What, Not How | Mutable variables, imperative validation | Type declarations, pure functions, system-enforced constraints |
| 11 | Rescue, Don't Rewrite | Big-bang rewrite | Extract one pure function, strangler pattern |
| 12 | Push Effects to the Edges | Business logic interleaved with I/O and side effects | Functional core (pure), imperative shell (I/O at boundaries) |

## Review Process

Run all steps for every review. Do not skip steps.

### 1) Build Evidence Per File

For each file in scope:

1. Read file (or diff hunk when scoped to changes).
2. Locate related tests (same module or nearby patterns).
3. Map code constructs against the cheat sheet above.
4. Note file size risk if file is large (500+ LOC).

### 2) Detect Dishonest Patterns

Map code against the "instead of" column of each construct. Each finding references a specific construct by name and number.

#### CRIME SCENE (High)

Active dishonest patterns causing real cost:

- Deep inheritance hierarchies (Construct 5)
- Hidden mutable state across modules (Construct 2)
- God classes bundling data and behavior (Construct 1)
- Mock-heavy tests that pass while integration fails (Construct 8)
- Dual state sources (client + server) that can diverge (Construct 3)
- Swallowed exceptions hiding real failures (Construct 6)
- Business logic functions that read from databases, call APIs, or write files mid-calculation (Construct 12)

#### SUSPECT (Medium)

Patterns trending toward dishonesty:

- Unnecessary indirection or wrapper layers (Construct 5)
- Premature caching without profiling evidence (Construct 7)
- Growing state surface area (Construct 3)
- Defensive error handling that masks root cause (Construct 6)
- Imperative logic where declarative would work (Construct 4, 10)
- Inline retry or fallback logic (Construct 6)
- Functions mixing calculation with logging or metrics emission (Construct 12)

#### WITNESS (Low)

Minor style-level issues:

- Clever expressions hiding simple intent (Construct 10)
- Single-implementation interfaces (Construct 5)
- Boilerplate exceeding logic (Construct 4)
- Imperative validation where type system could enforce (Construct 10)

### 3) Identify Honest Patterns

Call out code that follows the constructs well:

- Pure functions with clear input/output contracts (Construct 2)
- Flat data structures without behavior (Construct 1)
- Flat composition without inheritance (Construct 5)
- Boring tests: minimal setup, direct assertions (Construct 8)
- Declarative HTML/attributes over imperative DOM (Construct 4)
- Errors raised at source, handled at boundary (Construct 6)
- Single source of truth for state (Construct 3)
- Clear functional core / imperative shell separation (Construct 12)

### 4) Suggest Rescues

For each dishonest pattern found, propose the corresponding "use" alternative from the construct. Include language-appropriate code examples where helpful. Follow the strangler approach from Construct 11: suggest incremental changes, not rewrites.

## False-Positive Guardrails

- Do not flag patterns required by protocol, security, correctness, or framework constraints unless a simpler viable alternative is clear.
- Do not assume missing context; mark uncertain findings as uncertain.
- Prefer fewer high-confidence findings over many weak findings.
- Every finding must include specific file reference (`file:line`) and concrete symptom.
- If uncertain why code exists, say so and reference Chesterton's Fence.
- Do not claim certainty without evidence from code, tests, or explicit constraints.

## Output Contract

Use this structure unless user asked for a shorter variant:

```markdown
## Honest Code Review: [scope]

### Dishonest Pattern Sightings

**[file:line]** -- [CRIME SCENE|SUSPECT|WITNESS] Construct N: [construct name]
[What the code does]
[Why this pattern is dishonest]
[Rescue: honest alternative with code example if helpful]

### Honest Patterns Found

**[file:line]** -- Construct N: [what is honest and why]

### Verdict

- **Honesty level:** Honest / Mostly Honest / Mixed / Dishonest
- **Construct coverage:** [which constructs are relevant and how well they are followed]
- **Rescue priority:** [which construct violation to fix first and why]
- **One-sentence summary:** [single sentence]

### Suggested Rescues

1. [highest-impact rescue, referencing construct]
2. [next rescue]
3. [next rescue]

| Metric | Count |
|--------|-------|
| Files reviewed | N |
| CRIME SCENE | N |
| SUSPECT | N |
| WITNESS | N |
| Honest patterns | N |
| Honesty level | [level] |
```

If no issues found, say so explicitly and still provide the metric table.

## When This Skill Is Most Useful

- Before commit or PR
- During review of class hierarchies or mutable state
- When tests require extensive mock setup
- When state management feels duplicated
- When considering a rewrite vs. incremental rescue
- When evaluating AI-generated code for honesty
