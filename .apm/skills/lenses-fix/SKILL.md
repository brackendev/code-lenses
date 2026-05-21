---
name: lenses-fix
description: Apply non-conflicting fixes from default code lenses in parallel (APOSD and Legacy Code opt-in). Use --report to aggregate findings without writing.
argument-hint: [scope] [lenses] [+aposd|+legacy-code] [--report]
user-invocable: true
disable-model-invocation: true
---

# Lenses Fix

Run code lens reviews in parallel using the host runtime's sub-agent mechanism, aggregate findings into a unified report, then apply non-conflicting findings. By default, four lenses run. APOSD and Legacy Code are opt-in because they are situational (APOSD is most valuable for module boundary and interface design; Legacy Code is most valuable when code lacks tests or has hard dependencies).

This skill mutates by default. Pass `--report` to produce the aggregated findings report and skip the apply phase.

**Review Scope (optional):** "$ARGUMENTS"

## Arguments

Interpret naturally. This skill mutates by default. Pass `--report` to aggregate findings without writing.

| Input | Effect |
|-------|--------|
| (no argument) | Apply fixes from default four lenses to changed files (staged + unstaged) |
| `all` | Apply fixes from default four lenses across the full codebase (sampled for high-risk and high-traffic modules) |
| `<path>` `<glob>` | Apply fixes from default four lenses scoped to the path or pattern |
| `<lens-names>` | Run only the named lenses from `grug`, `honest-code`, `tidy-first`, `parse-dont-validate` |
| `+aposd` | Add the APOSD lens to the default set (documented exemption) |
| `+legacy-code` | Add the Legacy Code lens to the default set (documented exemption) |
| `--report` | Aggregate findings and print the report; skip the apply phase |

The `+aposd` and `+legacy-code` sigils are exemptions from the bare-keyword rule because a bare lens name selects a subset of the default lenses; the `+` distinguishes "add to defaults" from "subset of defaults". See CONVENTIONS.md exemptions for the rationale.

## Workflow

### 1. Determine Review Scope

Identify what to review:

- If the user provided a scope argument, use it as-is
- If no argument, run `git diff --name-only` and `git diff --cached --name-only` to identify changed files
- If no changed files and no argument, ask the user what to review
- If the arguments contain `--report`, enable report-only mode and remember to skip Step 5

Build a scope summary string (for example: "changed files: src/auth.ts, src/middleware.ts" or "path: src/api/") to pass to each sub-agent.

### 2. Available Review Lenses

**Default lenses (always run):**

| Lens | Focus |
|------|-------|
| `grug-review` | Complexity demons, premature abstraction, over-engineering |
| `honest-code-review` | Dishonest patterns, classes vs data, mutable state, inheritance |
| `tidy-first-review` | Structural tidying, mixed commits, reading order |
| `parse-dont-validate-review` | Type-driven correctness, boundary parsing, illegal states |

**Opt-in lenses (include by name):**

| Lens | Focus |
|------|-------|
| `aposd-review` | Module depth, information hiding, interface design |
| `legacy-code-review` | Missing tests, seams, safe modification of untested code |

### 3. Launch Review Sub-Agents

Launch the default four reviews **in parallel** using whatever sub-agent mechanism the host runtime provides. Each sub-agent receives the same scope summary.

For each lens:

- Start one sub-agent with a prompt telling it to apply exactly that review skill
- Use the prompt shape: "Review the following scope using the `[skill-name]` skill: [scope summary]. [any user modifiers]"
- Tell the sub-agent to return the full output contract for that lens
- Tell the sub-agent not to edit files

Start all requested sub-agents before waiting on any of them so the reviews run in parallel.

If the user specified a subset (for example: "grug honest-code"), launch only those lenses. If the user prefixes an opt-in lens with `+` (for example: "+aposd" or "+legacy-code"), add it to the default set rather than replacing it.

Each sub-agent invokes the review skill directly. Do not delegate to any packaged reviewer agent; no such agents are bundled with this plugin.

### 4. Aggregate Results

After all sub-agents complete, produce a unified report:

```markdown
## Lenses Fix: [scope]

### Verdicts

| Lens | Verdict | Summary |
|------|---------|---------|
| Grug Brain | [complexity demon power level] | [one-sentence summary] |
| Honest Code | [honesty level] | [one-sentence summary] |
| Tidy First? | [structural health level] | [one-sentence summary] |
| Parse Don't Validate | [type safety level] | [one-sentence summary] |
| APOSD (if included) | [module depth level] | [one-sentence summary] |
| Legacy Code (if included) | [change safety level] | [one-sentence summary] |

### Critical Findings

[High-severity findings from all lenses, grouped by file. Include the lens name and severity tier for each.]

### Other Findings

[Medium and low-severity findings, grouped by file.]

### Strengths

[Positive observations from all lenses.]

### Conflicts

[When two lenses give contradictory advice on the same code, list each conflict here. State both positions and which lens each comes from. Do not pick a winner. Known tension points:]

[- **Abstraction timing:** Grug delays abstraction until three repetitions. Tidy First extracts helpers whenever it eases the next change.]
[- **Type investment:** Grug wants the simplest type that works. Parse Don't Validate wants domain types that encode invariants. Resolution: use the cheapest type that permanently deletes a class of bugs.]
[- **Data design (when APOSD included):** Honest Code wants flat, transparent data. APOSD wants data hidden behind module interfaces.]
[- **Design investment (when APOSD included):** Grug favors shipping the simplest working solution. APOSD favors investing 10-20% extra time in strategic design.]
[- **Interface scope (when APOSD included):** Grug says design for current needs only. APOSD says design somewhat general-purpose interfaces.]
[- **Test strategy (when Legacy Code included):** Legacy Code says write characterization tests to lock existing behavior. Tidy First says tidy the structure before changing behavior. When both apply, decide which unblocks the change.]
[- **Seams vs composition (when Legacy Code included):** Legacy Code recommends object seams (subclass, Extract Interface) as tactical rescue techniques to enable testing. Honest Code avoids inheritance and deep hierarchies. These seams are temporary scaffolding for testability, not end-state design.]

[Only include conflicts that actually appeared in the review. Omit this section if no lenses contradicted each other.]

### Recommended Actions

1. [Highest-impact action across all lenses]
2. [Next action]
3. [Next action]
```

Deduplicate findings that overlap across lenses. When multiple lenses flag the same code, note the convergence. When lenses contradict each other on the same code, surface both positions in the Conflicts section and let the user decide.

### 5. Apply Fixes

Skipped when `--report` is active.

After presenting the unified report, apply the non-conflicting findings.

Launch a sub-agent with:
- The full unified report as context
- Instructions to apply all findings from Critical Findings and Other Findings
- Instructions to skip any finding listed in the Conflicts section (where lenses disagree)
- Instructions to prioritize Critical Findings over Other Findings
- Instructions to make minimal, targeted edits that address each finding

After the fix sub-agent completes, append a summary to the report:

```markdown
### Fixes Applied

| File | Change | Source Lens |
|------|--------|-------------|
| [file path] | [what changed] | [lens that suggested it] |

### Skipped (conflicting advice)

[Any findings skipped due to lens conflicts. Omit this section if none.]
```

## Usage Examples

**Full default run (apply fixes from default four lenses to changed files):**
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
```

**Add opt-in lenses to defaults:**
```text
lenses-fix +aposd
lenses-fix +legacy-code
lenses-fix +aposd +legacy-code src/services/
```

**Combine scope with report mode:**
```text
lenses-fix src/api/ --report
lenses-fix +aposd --report
```
