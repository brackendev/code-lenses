---
name: review-all
description: Run code lens reviews in parallel with optional fix (default 4, APOSD and Legacy Code opt-in)
argument-hint: [scope or options...]
allowed-tools: Agent, Bash, Read, Grep, Glob
user-invocable: true
disable-model-invocation: true
---

# Review All

Run code lens reviews in parallel using specialized agents, then aggregate findings into a unified report. Pass `fix` to apply non-conflicting findings after the review. By default, four lenses run. APOSD and Legacy Code are opt-in because they are situational (APOSD is most valuable for module boundary and interface design; Legacy Code is most valuable when code lacks tests or has hard dependencies).

**Review Scope (optional):** "$ARGUMENTS"

## Review Workflow

### 1. Determine Review Scope

Identify what to review:

- If the user provided a scope argument, use it as-is
- If no argument, run `git diff --name-only` and `git diff --cached --name-only` to identify changed files
- If no changed files and no argument, ask the user what to review
- If the arguments contain `fix`, enable fix mode and remove `fix` from the arguments before processing scope and lens selection

Build a scope summary string (for example: "changed files: src/auth.ts, src/middleware.ts" or "path: src/api/") to pass to each agent.

### 2. Available Review Lenses

**Default lenses (always run):**

| Agent | Lens | Focus |
|-------|------|-------|
| `grug-reviewer` | Grug brain | Complexity demons, premature abstraction, over-engineering |
| `honest-code-reviewer` | Honest Code | Dishonest patterns, classes vs data, mutable state, inheritance |
| `tidy-first-reviewer` | Tidy First? | Structural tidying, mixed commits, reading order |
| `parse-dont-validate-reviewer` | Parse Don't Validate | Type-driven correctness, boundary parsing, illegal states |

**Opt-in lenses (include by name):**

| Agent | Lens | Focus |
|-------|------|-------|
| `aposd-reviewer` | A Philosophy of Software Design | Module depth, information hiding, interface design |
| `legacy-code-reviewer` | Legacy Code | Missing tests, seams, safe modification of untested code |

### 3. Launch Review Agents

Launch the default four agents **in parallel** using the Agent tool. Each agent receives the same scope summary.

For each agent, set:
- `subagent_type` to the agent name (for example: `code-lenses:grug-reviewer`)
- The prompt to: "Review the following scope: [scope summary]. [any user modifiers]"

All Agent calls must be in a **single message** to run in parallel.

If the user specified a subset (for example: "grug honest-code"), launch only those agents. If the user prefixes an opt-in lens with `+` (for example: "+aposd" or "+legacy-code"), add it to the default set rather than replacing it.

### 4. Aggregate Results

After all agents complete, produce a unified report:

```markdown
## Review All: [scope]

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

[High-severity findings from all agents, grouped by file. Include the lens name and severity tier for each.]

### Other Findings

[Medium and low-severity findings, grouped by file.]

### Strengths

[Positive observations from all agents.]

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

### 5. Apply Fixes (when `fix` modifier is present)

After presenting the unified report, apply the non-conflicting findings.

Launch a general-purpose agent with:
- The full unified report as context
- Instructions to apply all findings from Critical Findings and Other Findings
- Instructions to skip any finding listed in the Conflicts section (where lenses disagree)
- Instructions to prioritize Critical Findings over Other Findings
- Instructions to make minimal, targeted edits that address each finding

After the fix agent completes, append a summary to the report:

```markdown
### Fixes Applied

| File | Change | Source Lens |
|------|--------|-------------|
| [file path] | [what changed] | [lens that suggested it] |

### Skipped (conflicting advice)

[Any findings skipped due to lens conflicts. Omit this section if none.]
```

## Usage Examples

**Full review (default):**
```
/review-all
```

**Specific scope:**
```
/review-all src/api/
/review-all src/auth.ts
/review-all all
```

**Subset of lenses:**
```
/review-all grug honest-code
/review-all tidy-first parse-dont-validate src/api/
```

**Add opt-in lenses to defaults:**
```
/review-all +aposd
/review-all +legacy-code
/review-all +aposd +legacy-code src/services/
```

**Review and fix:**
```
/review-all fix
/review-all fix src/api/
/review-all +aposd fix
```
