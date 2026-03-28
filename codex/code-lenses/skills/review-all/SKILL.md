---
name: review-all
description: Run all code lens reviews in parallel (grug, APOSD, Honest Code, Tidy First?)
argument-hint: [scope or options...]
user-invocable: true
disable-model-invocation: true
---

# Review All

Run all four code lens reviews in parallel using Codex sub-agents, then aggregate findings into a unified report.

**Review Scope (optional):** "$ARGUMENTS"

## Review Workflow

### 1. Determine Review Scope

Identify what to review:

- If the user provided a scope argument, use it as-is
- If no argument, run `git diff --name-only` and `git diff --cached --name-only` to identify changed files
- If no changed files and no argument, ask the user what to review

Build a scope summary string (for example: "changed files: src/auth.ts, src/middleware.ts" or "path: src/api/") to pass to each sub-agent.

### 2. Available Review Lenses

| Lens | Focus |
|------|-------|
| `grug-review` | Complexity demons, premature abstraction, over-engineering |
| `aposd-review` | Module depth, information hiding, interface design |
| `honest-code-review` | Dishonest patterns, classes vs data, mutable state, inheritance |
| `tidy-first-review` | Structural tidying, mixed commits, reading order |

### 3. Launch Review Sub-Agents

Launch all four reviews **in parallel** using Codex sub-agents. Each sub-agent receives the same scope summary.

For each lens:

- Start one sub-agent with a prompt telling it to apply exactly that review skill
- Use the prompt shape: "Review the following scope using the `[skill-name]` skill: [scope summary]. [any user modifiers]"
- Tell the sub-agent to return the full output contract for that lens
- Tell the sub-agent not to edit files

Start all requested sub-agents before waiting on any of them so the reviews run in parallel.

If the user specified a subset (for example: "grug aposd"), launch only those lenses.

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

### Critical Findings

[High-severity findings from all lenses, grouped by file. Include the lens name and severity tier for each.]

### Other Findings

[Medium and low-severity findings, grouped by file.]

### Strengths

[Positive observations from all lenses.]

### Recommended Actions

1. [Highest-impact action across all lenses]
2. [Next action]
3. [Next action]
```

Deduplicate findings that overlap across lenses. When multiple lenses flag the same code, note the convergence.

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
