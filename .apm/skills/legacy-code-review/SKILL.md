---
name: legacy-code-review
description: >-
  Review code for safe modification using Working Effectively with Legacy Code:
  seams, characterization tests, and breaking dependencies before changing
  untested code. Use when asked how to safely change or add tests to legacy
  code, or when the user mentions "legacy code", "seams", "characterization
  tests", or "add tests before refactoring". Read-only report.
argument-hint: "[scope or options...]"
user-invocable: true
---

# Legacy Code Review

Review code using techniques from [Working Effectively with Legacy Code](https://www.oreilly.com/library/view/working-effectively-with/0131177052/) by Michael Feathers. Identify missing test coverage, find seams for safe modification, and recommend characterization tests and incremental rescue strategies.

Legacy code is code without tests. This review finds paths to testability without rewriting.

All review output uses direct, professional voice.

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
| Focus | `seams only`, `test coverage`, `dependencies`, `entry points` |
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

## Legacy Code Checklist

Use this framework to evaluate code:

| Technique | What to Look For |
|-----------|-----------------|
| Seams | Points where behavior can be altered without editing the code: object seams (override via subclass or interface), link seams (swap dependency at build/config time), preprocessing seams (compile-time substitution). |
| Characterization tests | Tests that document existing behavior as-is, not as intended. Written before any change to detect unintended regressions. |
| Dependency breaking | Techniques to isolate code under test: Extract Interface, Parameterize Constructor, Extract and Override Call, Introduce Instance Delegator, Replace Global Reference with Getter. |
| Scratch refactoring | Temporary, throwaway refactoring to understand code. Check out a branch, make aggressive changes to learn the structure, then discard and apply targeted changes on the real branch. |
| Effect sketching | Tracing which methods and variables a change could affect. Drawing the dependency graph to find the narrowest test point. |
| Sprout method/class | Adding new behavior in a new method or class, called from the existing code. Avoids modifying untested code directly. |
| Wrap method/class | Wrapping existing behavior to add pre/post behavior without modifying the original. Useful when the original cannot be safely changed. |

## Review Process

Run all steps for every review. Do not skip steps.

### 1) Build Evidence Per File

For each file in scope:

1. Read file (or diff hunk when scoped to changes).
2. Search for related test files (same module, test directory, naming conventions).
3. Map dependencies: what does this code import, instantiate, or call? Which are hard dependencies (concrete classes, globals, static methods)?
4. Identify public entry points and internal coupling.
5. Estimate test coverage: are there tests? Do they cover the paths being changed?

### 2) Assess Legacy Risk

Evaluate each file or module:

1. **Test coverage:** Are there existing tests? Do they cover the code being changed?
2. **Dependency weight:** How many hard dependencies does this code have? Can it be instantiated in a test harness?
3. **Change risk:** Would modifying this code risk breaking behavior that is not covered by tests?
4. **Seam availability:** Are there existing seams (interfaces, dependency injection, configuration) that allow testing without modifying production code?

### 3) Apply Severity Tiers

#### UNTESTED (High)

Code with active risk due to missing test coverage:

- Business logic with no tests and upcoming changes planned
- Functions with complex branching (3+ paths) and no test coverage
- Shared utility code used across modules with no tests
- Code with hard dependencies that prevent isolation (global state, static calls, direct construction of heavy objects)
- Recent bug fixes applied without regression tests

#### BRITTLE (Medium)

Code trending toward legacy risk:

- Tests exist but do not cover the paths being changed
- Tests coupled to implementation details (mock-heavy, brittle assertions on internals)
- Hidden dependencies: code that reaches through layers to access globals or singletons
- Large methods or classes that resist extraction due to tangled state
- Configuration or environment dependencies baked into business logic

#### RIGID (Low)

Structural issues limiting future testability:

- No seams available: concrete classes instantiated directly, no interfaces
- Constructor does real work (I/O, network, database) making instantiation in tests expensive
- Static methods containing business logic that cannot be overridden
- Large parameter lists suggesting the function knows too much
- Tight coupling between modules that should be independent

### 4) Identify Safe Modification Patterns

Call out code that already follows legacy code techniques:

- Existing seams (interfaces, dependency injection) enabling isolated testing
- Characterization tests that document current behavior
- Sprout methods/classes adding new behavior without modifying untested code
- Wrapper patterns isolating new behavior from legacy code
- Clear dependency boundaries allowing test doubles

### 5) Recommend Rescue Strategy

For each finding, suggest a specific technique from the checklist:

- Which seam to exploit for testing
- What characterization test to write first
- Which dependency to break and how (Extract Interface, Parameterize Constructor, etc.)
- Whether to sprout, wrap, or modify in place
- The smallest safe first step

## False-Positive Guardrails

- Do not flag code as untested if tests exist in a non-obvious location. Search broadly before flagging.
- Do not recommend breaking dependencies that are stable, well-tested, and unlikely to change.
- Code without tests that is stable and never changes is lower priority than code without tests that is about to change.
- Do not assume missing context; mark uncertain findings as uncertain.
- Prefer fewer high-confidence findings over many weak findings.
- Every finding must include specific file reference (`file:line`) and concrete symptom.
- If uncertain why code exists, say so and reference Chesterton's Fence.
- Do not claim certainty without evidence from code, tests, or explicit constraints.

## Output Contract

Use this structure unless user asked for a shorter variant:

```markdown
## Legacy Code Review: [scope]

### Legacy Risk Findings

**[file:line]** -- [UNTESTED|BRITTLE|RIGID] [technique or principle]
[What the code does]
[Why this is a legacy risk]
[Rescue: specific technique with steps]

### Safe Modification Patterns Found

**[file:line]** -- [what pattern is in place and why it helps]

### Verdict

- **Test coverage:** Covered / Partially Covered / Sparse / Untested
- **Seam availability:** Rich / Adequate / Limited / None
- **Change safety:** Safe / Mostly Safe / Risky / Dangerous
- **One-sentence summary:** [single sentence]

### Rescue Strategy

1. [highest-impact rescue step, referencing technique]
2. [next step]
3. [next step]

| Metric | Count |
|--------|-------|
| Files reviewed | N |
| UNTESTED | N |
| BRITTLE | N |
| RIGID | N |
| Safe patterns | N |
| Change safety | [level] |
```

If no issues found, say so explicitly and still provide the metric table.

## When This Skill Is Most Useful

- Before modifying code that has no tests
- When inheriting an unfamiliar codebase
- When a bug fix needs a regression test but the code resists testing
- When planning how to add tests to a legacy module
- When deciding between modifying in place vs. sprouting new code
- When hard dependencies prevent instantiating code in a test harness
