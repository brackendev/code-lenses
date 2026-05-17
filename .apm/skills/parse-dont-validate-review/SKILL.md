---
name: parse-dont-validate-review
description: Review code for type-driven correctness using Parse Don't Validate and Make Illegal States Unrepresentable
argument-hint: [scope or options...]
user-invocable: true
disable-model-invocation: true
---

# Parse Don't Validate Review

Review code against [Parse, Don't Validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/) by Alexis King and [Make Illegal States Unrepresentable](https://blog.janestreet.com/effective-ml-revisited/) (coined by Yaron Minsky; see also [Scott Wlaschin's explainer](https://fsharpforfunandprofit.com/posts/designing-with-types-making-illegal-states-unrepresentable/)). Identify validation scattered through business logic, raw primitives used as domain concepts, and states modeled with booleans and nulls instead of sum types.

All review output uses direct, professional voice.

## Inputs and Scope

Interpret user input naturally:

| Input | Action |
|-------|--------|
| (no argument) | Review changed files only (staged + unstaged) |
| `path/to/dir` | Review files under directory |
| `path/to/file.ts` | Review specific file |
| `all` | Review full codebase (sample high-risk and high-traffic modules) |

Optional modifiers can appear anywhere in user input:

| Intent | Examples |
|--------|----------|
| Focus | `types only`, `boundaries`, `input handling`, `domain models` |
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

## Parse Don't Validate Checklist

Use this framework to evaluate code:

| Principle | What to Look For |
|-----------|-----------------|
| Parse at boundary | Raw input transformed into domain types at system edges (HTTP handlers, CLI, file readers, message consumers) |
| Illegal states unrepresentable | Sum types for mutually exclusive states. Branded/newtype for constrained primitives. No boolean flags modeling state machines. |
| Types carry proof | Functions accept parsed domain types, not raw strings/ints. Signature tells caller what is required. |
| Fail fast at boundary | Parse errors reported immediately at entry. No invalid data traveling deep into system. |
| No shotgun validation | Same invariant checked in one place (the parse function), not scattered across the codebase. |

## Review Process

Run all steps for every review. Do not skip steps.

### 1) Build Evidence Per File

For each file in scope:

1. Read file (or diff hunk when scoped to changes).
2. Identify system boundaries (HTTP handlers, CLI parsers, database queries, message consumers, file readers).
3. Map domain concepts: what data types represent domain entities and values?
4. Check whether domain types encode invariants or use raw primitives.
5. Trace how raw input flows from boundary into business logic.

### 2) Detect Validation Anti-Patterns

Map code against the anti-patterns below. Each finding references a specific principle.

#### UNGUARDED (High)

Active type-safety problems causing real cost:

- Raw primitives (`string`, `int`, `any`, `object`) used to represent domain concepts across module boundaries
- Same validation check repeated in 3+ locations (shotgun validation)
- States modeled with multiple booleans or nullable fields instead of sum types (e.g., `{ loading: boolean, error: string | null, data: T | null }`)
- Business logic functions accepting unvalidated input that could fail on invalid data
- No parse step at system boundaries: raw request/response bodies passed directly to business logic

#### LEAKING (Medium)

Patterns trending toward validation problems:

- Validation at boundary exists but returns the original unstructured type instead of a parsed domain type
- Optional fields used where a sum type would better represent the state space
- Partial parsing: some fields parsed into domain types, others left as raw primitives
- Type assertions or casts used instead of parsing (`as Email`, `(Email) input`)
- Error messages from deep in the system that belong at the boundary

#### LOOSE (Low)

Minor type-safety issues:

- Primitive obsession for a single concept (e.g., `userId: string` instead of `UserId`)
- Validation logic that could be a parse constructor but is not yet causing duplication
- Boolean parameters that could be a union type for clarity
- Missing exhaustiveness checks on discriminated unions or enums

### 3) Identify Parse-Driven Patterns

Call out code that follows the principles well:

- Domain types with parse constructors that enforce invariants (Parse at boundary)
- Sum types / discriminated unions modeling mutually exclusive states (Illegal states unrepresentable)
- Functions that accept only parsed types in their signatures (Types carry proof)
- Boundary handlers that parse all input before calling business logic (Fail fast at boundary)
- Single parse point per concept with no repeated validation downstream (No shotgun validation)
- Exhaustive pattern matching on sum types

### 4) Suggest Parses

For each anti-pattern found, propose the corresponding parse-driven alternative. Include language-appropriate code examples where helpful. Suggest incremental changes: introduce one domain type at a time, starting at the boundary.

## False-Positive Guardrails

- Do not flag raw primitives in internal utility functions where domain types would add overhead without safety benefit (e.g., string concatenation helpers).
- Do not flag dynamically-typed languages for missing branded types when the language has no type system. Suggest runtime parse functions instead.
- Do not assume missing context; mark uncertain findings as uncertain.
- Prefer fewer high-confidence findings over many weak findings.
- Every finding must include specific file reference (`file:line`) and concrete symptom.
- If uncertain why code exists, say so and reference Chesterton's Fence.
- Do not claim certainty without evidence from code, tests, or explicit constraints.

## Output Contract

Use this structure unless user asked for a shorter variant:

```markdown
## Parse Don't Validate Review: [scope]

### Validation Anti-Patterns

**[file:line]** -- [UNGUARDED|LEAKING|LOOSE] [principle violated]
[What the code does]
[Why this pattern is unsafe]
[Parse: type-driven alternative with code example if helpful]

### Parse-Driven Patterns Found

**[file:line]** -- [what is well-designed and which principle it follows]

### Verdict

- **Type safety:** Parsed / Mostly Parsed / Mixed / Unguarded
- **Boundary discipline:** Strong / Adequate / Weak / Absent
- **State modeling:** Sum types / Mostly sound / Boolean-heavy / Nullable
- **One-sentence summary:** [single sentence]

### Suggested Parses

1. [highest-impact parse, referencing principle]
2. [next parse]
3. [next parse]

| Metric | Count |
|--------|-------|
| Files reviewed | N |
| UNGUARDED | N |
| LEAKING | N |
| LOOSE | N |
| Parse-driven patterns | N |
| Type safety | [level] |
```

If no issues found, say so explicitly and still provide the metric table.

## When This Skill Is Most Useful

- When designing input handling or API boundaries
- When domain types use raw primitives (string IDs, untyped config)
- When the same validation appears in multiple places
- When state is modeled with booleans and nulls instead of unions
- When reviewing data flow from external systems into business logic
- When evaluating whether types encode the right invariants
