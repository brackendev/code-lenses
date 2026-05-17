---
name: aposd-review
description: Review code for module depth, information hiding, and complexity using A Philosophy of Software Design
argument-hint: [scope or options...]
user-invocable: true
disable-model-invocation: true
---

# APOSD Review

Review code against A Philosophy of Software Design principles by John Ousterhout. Identify shallow modules, information leakage, and complexity symptoms. Recommend deeper interfaces.

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
| Focus | `interfaces only`, `error handling`, `module boundaries`, `data layer` |
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

## Complexity Checklist

Use this framework to evaluate code:

| Principle | What to Look For |
|-----------|-----------------|
| Deep modules | Interface-to-implementation ratio. Simple interface hiding powerful functionality. |
| Information hiding | Design decisions encapsulated behind interfaces. Hidden data structures, algorithms, error recovery. |
| Information leakage | Same design decision appearing in multiple modules. Shared format/protocol knowledge. |
| Pull complexity down | Complexity in implementation vs interface. Configuration parameters pushing decisions to callers. |
| Errors out of existence | Interfaces redesigned so error conditions cannot occur. Idempotent operations. Exception aggregation. |
| General-purpose interfaces | Interfaces reflecting the domain, not specific use case requirements. Fewer special cases. |
| Strategic design | Small design investments per change. Incremental improvement over tactical shortcuts. |

## Review Process

Run all steps for every review. Do not skip steps.

### 1) Build Evidence Per File

For each file in scope:

1. Read file (or diff hunk when scoped to changes).
2. Map module boundaries: public interface vs hidden implementation.
3. Count interface surface area: public methods, parameters, configuration options.
4. Assess depth ratio: interface complexity relative to implementation power.
5. Note cross-module dependencies and shared design decisions.

### 2) Detect Complexity Symptoms

For each module, class, or significant abstraction:

1. **Change amplification:** Would a single-concept change (rename a field, change a format, modify a protocol) touch more than two modules?
2. **Cognitive load:** How much context from other modules is required to modify this one safely?
3. **Unknown unknowns:** Are there hidden dependencies, implicit contracts, or non-obvious side effects?

### 3) Apply Severity Tiers

#### SHALLOW (High)

Active complexity problems causing real cost:

- Pass-through methods that add no abstraction
- Modules where interface is more complex than implementation
- Information leakage across module boundaries (same format/protocol knowledge in multiple places)
- Configuration parameters pushing design decisions to callers
- Exception propagation through many layers without handling
- Change amplification: a single conceptual change requires touching 3+ modules

#### EXPOSED (Medium)

Patterns trending toward deeper complexity problems:

- Temporal decomposition (code split by execution order rather than information)
- Interfaces revealing implementation details (naming, parameter types, return shapes)
- Error handling distributed across layers instead of aggregated
- Growing cognitive load: understanding one module requires reading several others
- Shallow abstractions trending toward deeper leakage

#### SURFACE (Low)

Minor design-level issues:

- Interface could be slightly more general without added complexity
- Naming reveals implementation rather than purpose
- Default configuration values could reduce caller burden
- Small modules that could combine for a deeper abstraction

### 4) Identify Deep Design

Call out code that follows APOSD principles well:

- Deep modules with clean interface/implementation separation
- Information effectively hidden behind stable interfaces
- Errors defined out of existence or aggregated at a single level
- General-purpose interfaces that reflect the domain
- Strategic investment: design improvements alongside feature work
- Pull complexity downward: hard to implement, easy to use

## False-Positive Guardrails

- Do not flag shallow modules required by framework conventions (controller classes in MVC, route handlers, middleware signatures).
- Depth is relative to the problem domain. A three-line utility function is not shallow if the interface matches.
- Do not assume missing context; mark uncertain findings as uncertain.
- Prefer fewer high-confidence findings over many weak findings.
- Every finding must include specific file reference (`file:line`) and concrete symptom.
- If uncertain why code exists, say so and investigate before flagging.
- Do not claim certainty without evidence from code, tests, or explicit constraints.

## Output Contract

Use this structure unless user asked for a shorter variant:

```markdown
## APOSD Review: [scope]

### Complexity Symptoms

**[file:line]** -- [SHALLOW|EXPOSED|SURFACE] [principle violated]
[what the code does]
[which symptom: change amplification / cognitive load / unknown unknowns]
[recommendation: deeper interface, information hiding, error elimination, etc.]

### Deep Design Found

**[file:line]** -- [what is well-designed and which principle it follows]

### Verdict

- **Module depth:** Deep / Mostly Deep / Mixed / Shallow
- **Information hiding:** Strong / Adequate / Leaking / Exposed
- **Complexity trend:** Improving / Stable / Accumulating
- **One-sentence summary:** [single sentence]

### Suggested Improvements

1. [highest-impact improvement, referencing APOSD principle]
2. [next]
3. [next]

| Metric | Count |
|--------|-------|
| Files reviewed | N |
| SHALLOW | N |
| EXPOSED | N |
| SURFACE | N |
| Deep design | N |
| Module depth | [level] |
```

If no issues found, say so explicitly and still provide the metric table.

## When This Skill Is Most Useful

- When designing new modules or APIs
- When reviewing module boundary changes
- When error handling feels scattered across layers
- When a small change requires touching many files
- When inheriting a codebase and assessing design quality
- When deciding between deep and shallow decomposition
