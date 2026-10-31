---
name: lenses-fix
description: >-
  Detect the stack of the code in scope, choose the code lenses that fit it
  (grug, Honest Code, Tidy First, Parse Don't Validate, Data-Oriented
  Programming, APOSD, Legacy Code), run them in parallel, and apply the
  non-conflicting local fixes. Use when the user asks to run the code lenses
  and apply the results, or to fix the findings from a lens review. Writes
  fixes by default. Use --report to aggregate findings without writing.
argument-hint: "[scope] [lenses] [+lens] [apply architectural] [--report]"
user-invocable: true
---

# Lenses Fix

Identify the stack of each component in scope, choose the code lenses that fit each component, run the reviews in parallel using the host runtime's sub-agent mechanism, aggregate findings into a unified report, then apply the non-conflicting local findings. No lens list is fixed: `references/lens-selection.md` chooses lenses from what the code shows, and `references/stack-adaptation.md` tells every lens how to apply its source to the component's stack.

**Review Scope (optional):** "$ARGUMENTS"

## Arguments

Interpret naturally. This skill mutates by default. Pass `--report` to aggregate findings without writing.

| Input | Effect |
|-------|--------|
| (no argument) | Select lenses for the changed files (staged + unstaged) and apply their local fixes |
| `all` | Select lenses across the full codebase (sampled for high-risk and high-traffic modules) and apply their local fixes |
| `<path>` `<glob>` | Select lenses for the path or pattern and apply their local fixes |
| `<lens-names>` | Run only the named lenses from `grug`, `honest-code`, `tidy-first`, `parse-dont-validate`, `data-oriented-programming`, `aposd`, `legacy-code` |
| `+<lens>` | Add a lens to the selected set, for example `+aposd` or `+data-oriented-programming` (documented exemption) |
| `apply architectural` | Also apply the architectural findings in scope, which are otherwise reported only |
| `--report` | Aggregate findings and print the report; skip the apply phase |

A bare lens name runs only the named lenses, and any lens can be named. The `+` prefix adds a lens to the lenses that the selection rules choose.

## Scope

This skill excludes vendored, generated, and dependency-locked paths from the file set it walks. The filter combines `.gitignore` matches and a hardcoded floor (`node_modules/`, `vendor/`, `third_party/`, `.bundle/`, `target/`, `build/`, `dist/`, `out/`, `.shadow-cljs/`, `cljd-out/`, `*.lock`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `Gemfile.lock`, `Cargo.lock`, `poetry.lock`, `composer.lock`). Lens findings are never applied to code inside those paths under broad scopes. Naming a vendored path directly through `<path>` or `<glob>` bypasses the filter for that target. When the filter excludes any path, add a single "skipped N vendored or generated paths" line to the report. Under `--report`, list every skipped path.

## Workflow

### 1. Determine Review Scope

Identify what to review:

- If the user provided a scope argument, use it as-is
- If no argument, run `git diff --name-only` and `git diff --cached --name-only` to identify changed files
- If no changed files and no argument, ask the user what to review
- If the arguments contain `--report`, enable report-only mode and remember to skip Step 5
- If the arguments contain `apply architectural`, remember to apply architectural findings in Step 5

Build a scope summary string (for example: "changed files: src/auth.ts, src/middleware.ts" or "path: src/api/") to pass to each sub-agent.

### 2. Identify Components and Select Lenses

Read `references/stack-adaptation.md` and `references/lens-selection.md`.

1. Group the files in scope into components and record each component's language, framework, paradigm, typing, data representation, UI architecture, and test coverage, with file references (`stack-adaptation.md`, section 1).
2. Apply the selection rules in `lens-selection.md` to each component. Explicitly named lenses always run.
3. Keep the result for the report's Lens selection section: each component with its evidence, the lenses run and the rule that chose each, and the lenses not run with the reason.

| Lens | Review skill | Focus |
|------|--------------|-------|
| grug | `grug-review` | Complexity demons, premature abstraction, over-engineering |
| Honest Code | `honest-code-review` | Honest Code Principles P01-P25: I/O at the boundary, pure functions, faults, defaults, dispatch tables |
| Tidy First? | `tidy-first-review` | Structural tidying, mixed commits, reading order |
| Parse Don't Validate | `parse-dont-validate-review` | Boundary parsing, illegal states |
| Data-Oriented Programming | `data-oriented-programming-review` | Code and data separation, generic immutable data, separate schemas |
| APOSD | `aposd-review` | Module depth, information hiding, interface design |
| Legacy Code | `legacy-code-review` | Missing tests, seams, safe modification of untested code |

### 3. Launch Review Sub-Agents

Launch the selected reviews **in parallel** using whatever sub-agent mechanism the host runtime provides. Start all of them before waiting on any of them. Each sub-agent receives the scope summary and the component evidence for the files it reviews.

For each lens:

- Start one sub-agent with a prompt telling it to invoke exactly that review skill directly, not a separate reviewer agent
- Use the prompt shape: "Review the following scope using the `[skill-name]` skill: [scope summary]. Components: [component evidence]. Apply the lens to each component's stack following `lenses-fix/references/stack-adaptation.md`. [any user modifiers]"
- Tell the sub-agent to return the full output contract for that lens
- Tell the sub-agent not to edit files

Launch only the lenses selected in Step 2. When a lens runs for some components but not others, limit its scope summary to the files of the components it was selected for.

### 4. Aggregate Results

After all sub-agents complete, produce a unified report:

```markdown
## Lenses Fix: [scope]

### Lens Selection

| Component | Evidence | Lenses run (rule) | Not run (reason; how to run) |
|-----------|----------|-------------------|------------------------------|
| [name and files] | [language, framework, paradigm, typing, data representation, UI, tests, with file references] | [lens (rule from lens-selection.md)] | [lens (reason; `+name`)] |

### Verdicts

| Lens | Verdict | Summary |
|------|---------|---------|
| Grug Brain (if run) | [complexity demon power level] | [one-sentence summary] |
| Honest Code (if run) | [honesty level] | [one-sentence summary] |
| Tidy First? (if run) | [structural health level] | [one-sentence summary] |
| Parse Don't Validate (if run) | [type safety level] | [one-sentence summary] |
| Data-Oriented Programming (if run) | [data orientation level] | [one-sentence summary] |
| APOSD (if run) | [module depth level] | [one-sentence summary] |
| Legacy Code (if run) | [change safety level] | [one-sentence summary] |

### Critical Findings

[High-severity findings from all lenses, grouped by file. Include the lens name and severity tier for each.]

### Other Findings

[Medium and low-severity findings, grouped by file.]

### Architectural Departures

[Findings that would change a public contract, state ownership, persistence strategy, module responsibilities, or an established project convention, including every departure a lens reports because a framework or convention prevents its principle. Group them by design, one entry per design, not one per file: the design, the lenses and principles it departs from, the constraint or reason, and the incremental route where one exists (Honest Code P23 for moving from classes to functions). These are reported, not applied, unless the user passed `apply architectural`.]

### Strengths

[Positive observations from all lenses.]

### Conflicts

[When two lenses give contradictory advice on the same code, list each conflict here. State both positions and which lens each comes from. Do not pick a winner. An option that satisfies both lenses may be named, but the user decides. Known tension points:]

[- **Abstraction timing:** Grug delays abstraction until three repetitions. Tidy First extracts helpers whenever it eases the next change.]
[- **Type investment:** Grug wants the simplest type that works. Parse Don't Validate wants domain types that encode invariants. An option that satisfies both: use the cheapest type that permanently deletes a class of bugs.]
[- **Data shape:** Honest Code (P02) wants data as plain records with no methods. Parse Don't Validate wants constrained values wrapped in newtypes or branded types that only a parse function can create. A plain record produced only by a parse function satisfies both.]
[- **Data design (when APOSD included):** Honest Code (P02) wants plain, transparent data. APOSD wants data hidden behind module interfaces.]
[- **Design investment (when APOSD included):** Grug favors shipping the simplest working solution. APOSD favors investing 10-20% extra time in strategic design.]
[- **Interface scope (when APOSD included):** Grug says design for current needs only. APOSD says design somewhat general-purpose interfaces.]
[- **Test strategy (when Legacy Code included):** Legacy Code says write characterization tests to lock existing behavior. Tidy First says tidy the structure before changing behavior. When both apply, the user decides which unblocks the change.]
[- **Generic data vs domain types (when Data-Oriented Programming included):** Data-Oriented Programming (DOP #2, #4) keeps data in generic structures with a separate schema. Parse Don't Validate wants a distinct parsed type that carries the proof. In a dynamically typed component a parse function that returns schema-checked generic data satisfies both. In a statically typed component they disagree about the representation.]
[- **Open data vs information hiding (when Data-Oriented Programming and APOSD included):** DOP #1 lets any function read any data, a cost Sharvit names himself. APOSD hides data structures behind module interfaces.]
[- **Seams vs composition (when Legacy Code included):** Legacy Code recommends object seams (subclass, Extract Interface) as tactical rescue techniques to enable testing. Honest Code (P04) avoids inheritance. These seams are temporary scaffolding for testability, not end-state design, and P23 describes the same one-step-at-a-time migration.]

[Only include conflicts that actually appeared in the review. Omit this section if no lenses contradicted each other.]

### Recommended Actions

1. [Highest-impact action across all lenses]
2. [Next action]
3. [Next action]
```

Deduplicate findings that overlap across lenses. When multiple lenses flag the same code, note the convergence. List contradictions in the Conflicts section. Move every architectural finding and departure out of Critical Findings and Other Findings into Architectural Departures.

### 5. Apply Fixes

Skipped when `--report` is active.

After presenting the unified report, apply the non-conflicting local findings.

Some findings are reported instead of applied:

- **Mixed structural and behavioral changes** (Tidy First). The fix is to split the work into separate commits, which an edit cannot do.
- **Architectural Departures, from every lens.** A change to a public contract, state ownership, persistence strategy, module responsibilities, or an established project convention needs a decision by the people who own the design. Apply them only when the user passed `apply architectural`, or names specific findings in a later request, and then only within the requested scope.

Launch a sub-agent with:
- The full unified report as context
- Instructions to apply all findings from Critical Findings and Other Findings, plus Architectural Departures only when the user asked for them, except the report-only findings above
- Instructions to skip any finding listed in the Conflicts section (where lenses disagree)
- Instructions to apply findings in this order:
  1. Characterization tests from Legacy Code, when that lens ran, so existing behavior is locked before any other edit
  2. Structural changes that do not alter behavior (Tidy First tidyings, renames, extractions, dead code removal)
  3. Behavioral changes (new types, changed error handling, changed signatures), Critical Findings before Other Findings
- Instructions to make minimal, targeted edits that address each finding
- Instructions to classify each applied change as Test, Structural, or Behavioral

After the fix sub-agent completes, append a summary to the report:

```markdown
### Fixes Applied

| File | Change | Kind | Source Lens |
|------|--------|------|-------------|
| [file path] | [what changed] | [Test / Structural / Behavioral] | [lens that suggested it] |

[When both Structural and Behavioral changes were applied, state that the structural changes should be committed before the behavioral ones, in separate commits.]

### Reported, Not Applied

[Mixed-commit findings and Architectural Departures that were not applied, each with the lens and the recommended next step. Omit this section if none.]

### Skipped (conflicting advice)

[Any findings skipped due to lens conflicts. Omit this section if none.]

### Verification

[The command that ran the project's tests after the fixes, and its result. When the tests could not run, say why and name the command to run.]
```

After the fixes, run the project's test suite (or the narrowest command that covers the changed files) and report the result in the Verification section. Do not report the fixes as complete when tests fail. Report the failing tests instead.

## Usage Examples

**Full run (select lenses for the changed files and apply their local fixes):**
```text
lenses-fix
```

**Report only (no fixes applied):**
```text
lenses-fix --report
```

**Specific scope:**
```text
lenses-fix src/api/
lenses-fix src/auth.ts
lenses-fix all
```

**Subset of lenses:**
```text
lenses-fix grug honest-code
lenses-fix tidy-first parse-dont-validate src/api/
lenses-fix aposd src/services/
```

**Add lenses to the selection:**
```text
lenses-fix +aposd
lenses-fix +data-oriented-programming src/orders/
lenses-fix +aposd +legacy-code src/services/
```

**Apply architectural findings too:**
```text
lenses-fix src/orders/ apply architectural
```

**Combine scope with report mode:**
```text
lenses-fix src/api/ --report
lenses-fix +aposd --report
```
