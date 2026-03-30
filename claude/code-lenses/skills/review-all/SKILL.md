---
name: review-all
description: Run all code lens reviews in parallel (grug, APOSD, Honest Code, Tidy First?)
argument-hint: [scope or options...]
allowed-tools: Agent, Bash, Read, Grep, Glob
user-invocable: true
disable-model-invocation: true
---

# Review All

Run all four code lens reviews in parallel using specialized agents, then aggregate findings into a unified report.

**Review Scope (optional):** "$ARGUMENTS"

## Review Workflow

### 1. Determine Review Scope

Identify what to review:

- If the user provided a scope argument, use it as-is
- If no argument, run `git diff --name-only` and `git diff --cached --name-only` to identify changed files
- If no changed files and no argument, ask the user what to review

Build a scope summary string (for example: "changed files: src/auth.ts, src/middleware.ts" or "path: src/api/") to pass to each agent.

### 2. Available Review Lenses

| Agent | Lens | Focus |
|-------|------|-------|
| `grug-reviewer` | Grug brain | Complexity demons, premature abstraction, over-engineering |
| `aposd-reviewer` | A Philosophy of Software Design | Module depth, information hiding, interface design |
| `honest-code-reviewer` | Honest Code | Dishonest patterns, classes vs data, mutable state, inheritance |
| `tidy-first-reviewer` | Tidy First? | Structural tidying, mixed commits, reading order |

### 3. Launch Review Agents

Launch all four agents **in parallel** using the Agent tool. Each agent receives the same scope summary.

For each agent, set:
- `subagent_type` to the agent name (for example: `code-lenses:grug-reviewer`)
- The prompt to: "Review the following scope: [scope summary]. [any user modifiers]"

All four Agent calls must be in a **single message** to run in parallel.

If the user specified a subset (for example: "grug aposd"), launch only those agents.

### 4. Aggregate Results

After all agents complete, produce a unified report:

```markdown
## Review All: [scope]

### Verdicts

| Lens | Verdict | Summary |
|------|---------|---------|
| Grug Brain | [complexity demon power level] | [one-sentence summary] |
| APOSD | [module depth level] | [one-sentence summary] |
| Honest Code | [honesty level] | [one-sentence summary] |
| Tidy First? | [structural health level] | [one-sentence summary] |

### Critical Findings

[High-severity findings from all agents, grouped by file. Include the lens name and severity tier for each.]

### Other Findings

[Medium and low-severity findings, grouped by file.]

### Strengths

[Positive observations from all agents.]

### Conflicts

[When two lenses give contradictory advice on the same code, list each conflict here. State both positions and which lens each comes from. Do not pick a winner. Known tension points:]

[- **Data design:** Honest Code wants flat, transparent data. APOSD wants data hidden behind module interfaces.]
[- **Abstraction timing:** Grug delays abstraction until three repetitions. Tidy First extracts helpers whenever it eases the next change.]
[- **Design investment:** Grug favors shipping the simplest working solution. APOSD favors investing 10-20% extra time in strategic design.]
[- **Interface scope:** Grug says design for current needs only. APOSD says design somewhat general-purpose interfaces.]

[Only include conflicts that actually appeared in the review. Omit this section if no lenses contradicted each other.]

### Recommended Actions

1. [Highest-impact action across all lenses]
2. [Next action]
3. [Next action]
```

Deduplicate findings that overlap across lenses. When multiple lenses flag the same code, note the convergence. When lenses contradict each other on the same code, surface both positions in the Conflicts section and let the user decide.

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
/review-all grug aposd
/review-all tidy-first honest-code src/api/
```
