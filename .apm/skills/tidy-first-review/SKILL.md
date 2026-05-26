---
name: tidy-first-review
description: Review code for tidying opportunities using Tidy First? philosophy
argument-hint: "[scope or options...]"
user-invocable: true
disable-model-invocation: true
---

# Tidy First? Review

Review code for tidying opportunities using the [Tidy First?](https://www.oreilly.com/library/view/tidy-first/9781098151232/) philosophy by Kent Beck (O'Reilly, 2023). Identify structural changes that would make behavioral changes safer and easier.

All review output uses direct, professional voice. Reference tidyings by name and number.

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
| Focus | `guard clauses`, `dead code`, `reading order`, `mixed commits` |
| Output shape | `verdict only`, `no approvals`, `problems only` |

Always run the full review flow. Do not provide reduced-depth modes.

## Context Gathering

Run these commands for scope discovery:

- `git status --short`
- `git diff --name-only`
- `git diff --cached --name-only`
- `git log --oneline -10` — recent commits for mixed-commit detection
- `git log --oneline --diff-filter=M -10` — recent commits that modified files

For explicit scope arguments, collect files from provided paths.
Filter to source/test/configuration files relevant to behavior. Skip generated files, lock files, vendored dependencies, and binaries.

If no files are found (and no explicit scope), ask user what to review.

## The 15 Tidyings (Cheat Sheet)

Use this framework to evaluate code. Each tidying defines a signal (what to look for) and an action (what to do).

| # | Tidying | Signal | Action |
|---|---------|--------|--------|
| 1 | Guard clauses | Nested if-else, deep indentation | Replace with early returns, flatten control flow |
| 2 | Explaining variables | Complex expressions inline | Extract into named variables that reveal intent |
| 3 | Explaining constants | Magic numbers or strings | Replace with named constants |
| 4 | Explicit parameters | Globals, configuration lookups, implicit state | Pass as function parameters |
| 5 | Chunk statements | Long runs of code with no visual separation | Group related statements with blank lines |
| 6 | Extract helper | Cohesive block doing one thing inside a larger function | Pull into a named function |
| 7 | One pile | Overly fragmented code spread across many small functions | Inline back together, then re-extract with better boundaries |
| 8 | Dead code | Unreachable branches, unused functions, commented-out code | Delete after verifying with search |
| 9 | Normalize symmetries | Similar code using different structural patterns | Make patterns identical so differences stand out |
| 10 | New interface, old implementation | Awkward interface forcing callers into workarounds | Create the interface you want, delegate to existing code |
| 11 | Reading order | Declarations out of sequence, callees before callers | Reorder top-down so readers encounter definitions in use order |
| 12 | Cohesion order | Related code scattered across the file | Move related functions and data closer together |
| 13 | Move declaration and initialization together | Variables declared far from first use | Declare where first used, not at top of scope |
| 14 | Remove unnecessary comments | Comments restating what the code says | Delete redundant comments |
| 15 | Eliminate needless complexity | Abstractions, parameters, or indirection with no current purpose | Remove what serves no current need |

## Review Process

Run all steps for every review. Do not skip steps.

### 1) Build Evidence Per File

For each file in scope:

1. Read file (or diff hunk when scoped to changes).
2. Locate related tests (same module or nearby patterns).
3. Map code against the 15-tidying cheat sheet.
4. Note file size risk if file is large (500+ lines of code).

### 2) Detect Mixed Changes

Primary check: are structural and behavioral changes separated?

**When reviewing commits** (recent history or PR): Use `git show <sha>` to inspect each commit. A commit that contains both structural moves (renaming, reordering, extracting) and behavioral changes (new logic, changed conditions, different return values) is always TANGLED.

**When reviewing uncommitted changes** (staged + unstaged diff): Scan diff hunks for regions that mix structural and behavioral changes in the same file or logical unit. Flag hunks where tidyings and new behavior are interleaved. Recommend splitting into separate commits before pushing.

### 3) Detect Tidying Opportunities

Map code against tidying signals. Classify by severity:

#### TANGLED (High)

Active structural problems causing real cost:

- Mixed structural and behavioral changes in same commit
- Deep nesting (3+ levels) where guard clauses would flatten (Tidying 1)
- Scattered related code requiring readers to jump across file sections (Tidying 12)
- Dead code blocking understanding of live code paths (Tidying 8)

#### CLUTTERED (Medium)

Patterns trending toward structural debt:

- Magic numbers or strings without explaining constants (Tidying 3)
- Complex expressions without explaining variables (Tidying 2)
- Declarations far from first use (Tidying 13)
- Comments restating what the code says (Tidying 14)
- Similar code with unnecessary structural differences (Tidying 9)
- Functions doing multiple unrelated things (Tidying 6)

#### DUSTY (Low)

Minor structural issues:

- Reading order could be improved, callers after callees (Tidying 11)
- Cohesion order could be improved, related functions far apart (Tidying 12)
- Implicit parameters where explicit would clarify (Tidying 4)
- Minor redundant complexity that could be eliminated (Tidying 15)

### 4) Identify Good Structure

Call out code that follows Tidy First principles well:

- Guard clauses and early returns keeping control flow flat (Tidying 1)
- Named intermediate variables revealing intent (Tidying 2)
- Constants for magic values (Tidying 3)
- Clean separation of structural and behavioral commits
- Code in reading order, callers before callees (Tidying 11)
- Related code grouped together (Tidying 12)
- Focused functions doing one thing (Tidying 6)

## False-Positive Guardrails

- Do not flag patterns required by framework or language constraints unless a simpler viable alternative is clear.
- Do not recommend tidying code that is not about to change (speculative tidying).
- Do not assume missing context; mark uncertain findings as uncertain.
- Prefer fewer high-confidence findings over many weak findings.
- Every finding must include specific file reference (`file:line`) and concrete symptom.
- If uncertain why code exists, say so and reference Chesterton's Fence.
- Do not claim certainty without evidence from code, tests, or explicit constraints.

## Output Contract

Use this structure unless user asked for a shorter variant:

```markdown
## Tidy First? Review: [scope]

### Mixed Commit Findings

**[file:line]** -- [TANGLED] [description]
[what is structural vs behavioral]
[recommendation: split into separate commits]

### Tidying Opportunities

**[file:line]** -- [TANGLED|CLUTTERED|DUSTY] Tidying N: [name]
[current structure]
[why this hurts the next behavioral change]
[specific tidying to apply]

### Good Structure

**[file:line]** -- [what is well-structured and why]

### Verdict

- **Structural health:** Clean / Mostly Clean / Needs Tidying / Tangled
- **Mixed commits:** [count or "none detected"]
- **Tidying priority:** [which tidying to apply first and why]
- **One-sentence summary:** [single sentence]

### Suggested Tidyings

1. [highest-impact tidying]
2. [next]
3. [next]

| Metric | Count |
|--------|-------|
| Files reviewed | N |
| TANGLED | N |
| CLUTTERED | N |
| DUSTY | N |
| Good structure | N |
| Structural health | [level] |
```

If no issues found, say so explicitly and still provide the metric table.

## When This Skill Is Most Useful

- Before commit or PR
- When inheriting unfamiliar code
- When a behavioral change feels harder than it should
- During review of refactoring PRs
- When deciding whether to tidy first, after, later, or never
