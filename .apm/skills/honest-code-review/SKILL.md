---
name: honest-code-review
description: >-
  Review code for dishonest patterns using the Honest Code Principles
  (P01-P25): I/O inside business logic, methods that mutate state, swallowed
  faults, implicit defaults, mock-heavy tests, and unchecked dispatch. Use
  when asked for an "honest code review", or when the user mentions
  "dishonest", "hidden side effects", "pure functions", or "is this function
  honest". Read-only report.
argument-hint: "[scope or options...]"
user-invocable: true
---

# Honest Code Review

Review code against the Honest Code Principles (P01-P25), maintained by the Open Honest Foundation (Adam Zachary Wasserman). The rule text is the unchanged upstream file that the `honest-code` skill ships at a pinned commit: `honest-code/references/honest-code-principles.md`, next to this skill's directory. Read that file before reviewing. It is the rule text, and this skill cites it by number without restating it. The examples below are short pointers to common violations, not definitions, so read the cited principle before reporting a finding. Upstream asks projects not to copy the file and not to summarize it, because copies drift. This package ships the exact file anyway so that reviews work without network access. Identify dishonest patterns and suggest rescues.

All review output uses direct, professional voice. Cite each principle by number (`P15`), never by title alone, because upstream keeps numbers permanently and has renamed titles.

Apply the lens to the reviewed code's stack following `lenses-fix/references/stack-adaptation.md`, next to this skill's directory: identify each component's language, paradigm, and typing from the code, translate the mechanism while keeping the prescription, never recommend simulating a feature the language lacks, and report a framework or convention conflict as a departure, never as conformance.

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

Output-shape modifiers change what the report shows, not how much of the scope the review covers.

## Context Gathering

Run these commands for scope discovery:

- `git status --short`
- `git diff --name-only`
- `git diff --cached --name-only`

For explicit scope args, collect files from provided paths.
Filter to source/test/config files relevant to behavior. Skip generated files, lock files, vendored dependencies, and binaries.

If no files are found (and no explicit scope), ask user what to review.

## Applying the Principles

- Evaluate the code against every principle in the file. Each principle names a category of defect, so a finding names that defect in the reviewed code.
- P05 (DOM as State), P06 (HTML attributes), and P22 (declarative lifecycle) concern browser interfaces. When the scope has no browser interface, list them as not applicable. When it has one built on another client-state design, report the disagreement.
- When a framework or project constraint prevents a principle (for example, ORM entity classes or a dependency-injection container), report the code as a departure with its reason. Never report a departure as conformance, and never restate the principle in weaker terms to make the code pass.
- The file's "Enforced by" lines describe upstream's own checkers. Do not report that a project is checked by them unless it runs them.

## Review Process

### 1) Build Evidence Per File

For each file in scope:

1. Read file (or diff hunk when scoped to changes).
2. Locate related tests (same module or nearby patterns).
3. Compare the code with each principle in the principles file.
4. Note file size risk if file is large (500+ LOC).

### 2) Detect Dishonest Patterns

The principles do not rank themselves. The three severity tiers below are this package's review scale, ranked by the cost the pattern causes in the reviewed code, and the examples are common sightings, not a complete list. Each example points to a principle and does not replace its text. A pattern that matches a principle and appears in no example is still a finding.

#### CRIME SCENE (High)

Active dishonest patterns causing real cost:

- Business logic that reads databases, calls APIs, or writes files mid-calculation (P03)
- Methods that mutate shared instance state, or classes bundling data with behavior across modules (P02)
- Deep inheritance hierarchies, or orchestrators that call other orchestrators (P04)
- Exceptions caught and swallowed in business logic, or faults that leave a function outside its return value (P08, P24)
- Mock-heavy tests that pass while integration fails (P10)
- Check-then-act guards on shared state (P17)
- Client state that shadows server state and can diverge, when the scope has a browser interface (P05)

#### SUSPECT (Medium)

Patterns trending toward dishonesty:

- if/elif or switch chains used for dispatch (P01)
- Dispatch tables read with a default instead of by key, so an unknown key passes as a handled one (P16)
- Implicit default parameters (P15)
- Caches added without profiling evidence (P09)
- Inline retry or fallback logic in business functions (P08)
- Functions that log errors instead of returning them, or call a global logger directly (P18)
- Configuration or resources stored on instances (P12, P13)
- Imperative DOM wiring or lifecycle hooks, when the scope has a browser interface (P06, P22)

#### WITNESS (Low)

Minor issues:

- Defensive checks in the interior that re-test what the signature already promises (P14)
- An isolated hand-written validation check where a declared schema or type could enforce the constraint (P11)
- Fault messages that name a state without saying what to do differently (P25)

### 3) Identify Honest Patterns

Call out code that follows the principles well, citing the number: for example, lookup dispatch (P01), pure functions over plain data (P02), I/O only at the edges (P03), visible pipelines (P04), and tests that assert on a pure function's output (P10).

### 4) Suggest Rescues

For each dishonest pattern, propose the honest alternative that the cited principle gives. Include language-appropriate code examples where helpful. Follow P23: suggest incremental changes, one extracted function at a time, not rewrites.

## False-Positive Guardrails

- Do not flag patterns required by protocol, security, correctness, or framework constraints unless a simpler viable alternative is clear.
- Do not assume missing context or claim certainty without evidence from code, tests, or explicit constraints. Mark uncertain findings as uncertain.
- Prefer fewer high-confidence findings over many weak findings.
- Every finding must include specific file reference (`file:line`) and concrete symptom.
- If uncertain why code exists, say so and reference Chesterton's Fence.

## Output Contract

Use this structure unless user asked for a shorter variant:

```markdown
## Honest Code Review: [scope]

### Dishonest Pattern Sightings

**[file:line]** -- [CRIME SCENE|SUSPECT|WITNESS] PNN: [principle title]
[What the code does]
[The defect category the principle names, as it appears here]
[Rescue: honest alternative with code example if helpful]

### Honest Patterns Found

**[file:line]** -- PNN: [what is honest and why]

### Verdict

- **Honesty level:** Honest / Mostly Honest / Mixed / Dishonest
- **Principle coverage:** [which principles are relevant and how well they are followed]
- **Not applicable:** [principles that do not apply to this scope, with the reason, or "none"]
- **Departures:** [code that departs from a principle because of a framework or project constraint, with the constraint, or "none"]
- **Rescue priority:** [which violation to fix first and why]
- **One-sentence summary:** [single sentence]

### Suggested Rescues

1. [highest-impact rescue, citing the principle number]
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
