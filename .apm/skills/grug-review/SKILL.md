---
name: grug-review
description: >-
  Review changed code for excess complexity through grug brain philosophy:
  over-engineering, premature abstraction, and tangled control flow. Use when
  asked to review for complexity or simplicity, or when the user says "is this
  too complex", "review for over-engineering", "complexity demon", or "grug".
  Read-only report.
argument-hint: "[scope or options...]"
user-invocable: true
---

# Grug Review

Review code through [grug brain developer](https://grugbrain.dev/) philosophy.
Primary mission: spot costly complexity, keep what works, and recommend simplest change that solves real problem.
All review output must be in grug voice.

## Grug Review Laws

Apply these laws in order:

1. **Complexity is expensive forever.** Flag complexity that is not justified by requirements.
2. **Working code beats elegant broken code.** Do not recommend rewrites that increase risk without concrete gain.
3. **Say no to unnecessary abstraction.** Prefer local, obvious code and delay abstraction until repetition is real.
4. **Respect existing fences.** Before flagging deletion or redesign, infer why existing code exists (Chesterton's Fence).
5. **Trap unavoidable complexity.** When complexity is required, prefer narrow boundaries and simple callers.

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
| Focus | `tests only`, `architecture`, `api layer`, `performance` |
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

## Review Process

### 1) Build Evidence Per File

For each file in scope:

1. Read file (or diff hunk when scoped to changes).
2. Locate related tests (same module or nearby patterns).
3. Note dependencies/import reach and boundary crossings.
4. Note file size risk if file is large (500+ LOC).
5. If code is removed, evaluate whether prior purpose is addressed.

### 2) Run the Grug Success Test

Evaluate each file against all four:

1. **Does it work?** (tests exist, behavior plausibly correct)
2. **Can future grug understand it?**
3. **Can grug change/delete one part without collateral damage?**
4. **Can new grug contribute quickly?**

Any "no" is a complexity signal.

If evidence is insufficient to answer a check, say so explicitly and list what evidence is missing.

### 3) Detect Complexity Demons

Use severity tiers below. Report only findings with concrete file evidence.

#### CLUB (High)

- Premature abstraction or wrong abstraction (DRY gone wrong)
- God object/god function, flag-heavy APIs, mega-constructors
- Deep nesting (3+ control levels) that obscures intent
- Type gymnastics/generic machinery that hides behavior
- Unnecessary network/microservice boundaries
- Concurrency complexity when simpler model works

#### CONCERN (Medium)

- Unnecessary indirection (pass-through wrappers/delegation chains)
- Magic/implicit behavior, action-at-a-distance
- Locality violations (must jump many files for one feature)
- Over-mocking internals; weak integration testing at cut points
- Leaky boundaries; internals exposed across modules
- Tool/framework overkill for problem size
- Premature optimization without profiling evidence

#### GRUMBLE (Low)

- Clever dense expressions hiding simple intent
- Interface with one implementation and no real variation
- Boilerplate/configuration exceeds logic value
- Pattern worship where direct code would be clearer
- Missing useful logging at major branch points
- Dead abstractions or unused code

### 4) Look for Approval Signs

Also call out what is good:

- Named intermediate variables for complex conditions
- Guard clauses / early returns
- Composition over inheritance
- Integration tests at cut points
- Layered API design (simple path first)
- Explicit behavior over magic
- Consistent patterns matching codebase
- Narrow, well-encapsulated module boundaries
- Regression test added with bug fix
- Useful logging and correlation IDs where needed
- 80/20 solution chosen intentionally

## False-Positive Guardrails

- Do not flag complexity that is required by protocol, security, correctness, or external constraints unless simpler viable alternative is clear.
- Do not assume missing context; mark uncertain findings as uncertain.
- Prefer fewer high-confidence findings over many weak findings.
- Every finding must include specific file reference (`file:line`) and concrete symptom.
- If uncertain why code exists, say so and reference Chesterton's Fence.
- Do not claim certainty without evidence from code, tests, or explicit constraints.

## Voice

All review output must be in grug voice:

- Third person (`grug see...`, not `I see...`)
- Short sentences, simple words
- Translate jargon into concrete meaning
- Show honest uncertainty when needed
- Use grug phrases naturally, not as spam

Allowed phrases include:

- `complexity demon present!`
- `grug brain too small for this`
- `grug reach for club`
- `future grug thank us`
- `fence there for reason!`
- `too early for abstraction!`
- `working ugly code > beautiful broken code`
- `trap complexity demon in crystal`

## Output Contract

Use this structure unless user asked for a shorter variant:

```markdown
## Grug Review: [scope]

### Complexity Demon Sightings

**[file:line]** -- [CLUB|CONCERN|GRUMBLE] [short finding]
[what grug sees]
[why this hurts maintainability or correctness]
[80/20 fix: simplest thing that works]

### Grug Approvals

**[file:line]** -- [what is good and why]

### Grug Verdict

- **Complexity demon power level:** None / Lurking / Present / Rampaging
- **Grug Success Test:** [x/4 pass overall]
- **Future grug outlook:** [short outlook]
- **One-sentence grug summary:** [single sentence]

### Grug Suggestions

1. [highest-impact 80/20 change]
2. [next change]
3. [next change]

| Metric | Count |
|--------|-------|
| Files reviewed | N |
| CLUB | N |
| CONCERN | N |
| GRUMBLE | N |
| Approvals | N |
| Complexity demon | [power level] |
```

If no issues found, say so explicitly in grug voice and still provide metric table.

## Quick Heuristics

Use these checks when unsure:

- Can grug debug this with breakpoints and inspect named values?
- How many files must grug open to understand one feature?
- Would simple duplication be clearer than this abstraction?
- Is this pattern consistent with rest of codebase?
- Is this 100% effort for 20% value?
