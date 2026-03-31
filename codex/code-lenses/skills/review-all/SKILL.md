---
name: review-all
description: Run code lens reviews in parallel (default 5, Legacy Code opt-in)
argument-hint: [scope or options...]
user-invocable: true
disable-model-invocation: true
---

# Review All

Run code lens reviews in parallel using Codex sub-agents, then aggregate findings into a unified report. By default, five lenses run. Legacy Code is opt-in because it is situational (most valuable when code lacks tests or has hard dependencies).

**Review Scope (optional):** "$ARGUMENTS"

## Review Workflow

### 1. Determine Review Scope

Identify what to review:

- If the user provided a scope argument, use it as-is
- If no argument, run `git diff --name-only` and `git diff --cached --name-only` to identify changed files
- If no changed files and no argument, ask the user what to review

Build a scope summary string (for example: "changed files: src/auth.ts, src/middleware.ts" or "path: src/api/") to pass to each sub-agent.

### 2. Available Review Lenses

**Default lenses (always run):**

| Lens | Focus |
|------|-------|
| `grug-review` | Complexity demons, premature abstraction, over-engineering |
| `aposd-review` | Module depth, information hiding, interface design |
| `honest-code-review` | Dishonest patterns, classes vs data, mutable state, inheritance |
| `tidy-first-review` | Structural tidying, mixed commits, reading order |
| `parse-dont-validate-review` | Type-driven correctness, boundary parsing, illegal states |

**Opt-in lenses (include by name):**

| Lens | Focus |
|------|-------|
| `legacy-code-review` | Missing tests, seams, safe modification of untested code |

### 3. Launch Review Sub-Agents

Launch the default five reviews **in parallel** using Codex sub-agents. Each sub-agent receives the same scope summary.

For each lens:

- Start one sub-agent with a prompt telling it to apply exactly that review skill
- Use the prompt shape: "Review the following scope using the `[skill-name]` skill: [scope summary]. [any user modifiers]"
- Tell the sub-agent to return the full output contract for that lens
- Tell the sub-agent not to edit files

Start all requested sub-agents before waiting on any of them so the reviews run in parallel.

If the user specified a subset (for example: "grug aposd"), launch only those lenses. If the user prefixes an opt-in lens with `+` (for example: "+legacy-code"), add it to the default set rather than replacing it.

Do not rely on packaged reviewer agents in the Codex copy. The Codex version uses the review skills directly.

### 4. Aggregate Results

After all sub-agents complete, produce a unified report:

```markdown
## Review All: [scope]

### Verdicts

| Lens | Verdict | Summary |
|------|---------|---------|
| Grug Brain | [complexity demon power level] | [one-sentence summary] |
| APOSD | [module depth level] | [one-sentence summary] |
| Honest Code | [honesty level] | [one-sentence summary] |
| Tidy First? | [structural health level] | [one-sentence summary] |
| Parse Don't Validate | [type safety level] | [one-sentence summary] |
| Legacy Code (if included) | [change safety level] | [one-sentence summary] |

### Critical Findings

[High-severity findings from all lenses, grouped by file. Include the lens name and severity tier for each.]

### Other Findings

[Medium and low-severity findings, grouped by file.]

### Strengths

[Positive observations from all lenses.]

### Conflicts

[When two lenses give contradictory advice on the same code, list each conflict here. State both positions and which lens each comes from. Do not pick a winner. Known tension points:]

[- **Data design:** Honest Code wants flat, transparent data. APOSD wants data hidden behind module interfaces.]
[- **Abstraction timing:** Grug delays abstraction until three repetitions. Tidy First extracts helpers whenever it eases the next change.]
[- **Design investment:** Grug favors shipping the simplest working solution. APOSD favors investing 10-20% extra time in strategic design.]
[- **Interface scope:** Grug says design for current needs only. APOSD says design somewhat general-purpose interfaces.]
[- **Type investment:** Grug wants the simplest type that works. Parse Don't Validate wants domain types that encode invariants.]
[- **Test strategy:** Legacy Code says write characterization tests to lock existing behavior. Tidy First says tidy the structure before changing behavior. When both apply, decide which unblocks the change.]
[- **Seams vs composition:** Legacy Code recommends object seams (subclass, Extract Interface) as tactical rescue techniques to enable testing. Honest Code avoids inheritance and deep hierarchies. These seams are temporary scaffolding for testability, not end-state design.]

[Only include conflicts that actually appeared in the review. Omit this section if no lenses contradicted each other.]

### Recommended Actions

1. [Highest-impact action across all lenses]
2. [Next action]
3. [Next action]
```

Deduplicate findings that overlap across lenses. When multiple lenses flag the same code, note the convergence. When lenses contradict each other on the same code, surface both positions in the Conflicts section and let the user decide.

## Usage Examples

**Full review (default):**
```text
review-all
```

**Specific scope:**
```text
review-all src/api/
review-all src/auth.ts
review-all all
```

**Subset of lenses:**
```text
review-all grug aposd
review-all tidy-first honest-code src/api/
```

**Add Legacy Code to defaults:**
```text
review-all +legacy-code
review-all +legacy-code src/services/
```
