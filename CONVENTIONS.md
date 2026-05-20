# Conventions

This document defines the canonical argument grammar, scope vocabulary, and mutation default for user-invocable skills in this plugin. A reader who learns one skill should be able to predict the argument shape and runtime behavior of every other skill.

Model-auto-triggered skills (the five implementation-guidance lenses `aposd`, `grug`, `honest-code`, `parse-dont-validate`, `tidy-first`) are out of scope. They have no user-facing argument surface.

## Three Rules

### Rule 1: Argument grammar

User-invocable skills accept natural-language keywords and bare paths. The single sanctioned flag is `--report`. No other `--name` flags exist. Skill-specific modifiers are bare phrases (`tests only`, `verdict only`, `problems only`). Document any exemption with a reason. The recognized exemptions in this plugin are listed in the Exemptions section below.

### Rule 2: Scope vocabulary

File-aware skills use the same core rows in their Arguments table:

| Input             | Target                                                     |
|-------------------|------------------------------------------------------------|
| (no argument)     | The skill's narrowest useful default                       |
| `all`             | Widen the selected scope to its maximum                    |
| `<path>` `<glob>` | Operate on those files or directories                      |

Opt-in rows appear only when the skill genuinely supports them:

| Input             | Target                                                     |
|-------------------|------------------------------------------------------------|
| `#N` or PR URL    | That pull request                                          |
| `pr`              | Current branch's pull request title and body               |
| `commit`          | Most recent commit message plus staged draft               |

The `all` keyword has one uniform meaning across skills: widen the selected scope to the maximum. The concrete unit varies per skill (whole codebase, every detected stack, every recent commit) and is stated in each skill's Arguments table. Skills with no narrower default than maximum scope accept `all` for family consistency.

### Rule 3: Mutation is the default

A skill that can mutate the workspace applies its changes when invoked. The operator passes `--report` to receive a description of what the skill would do without modifying any files. Only the literal token `--report` enables report-only mode; natural-language synonyms ("preview", "dry run") are scope input, not mode triggers.

Command suffixes reinforce the default. The family follows a noun-first `<target>-<verb>` pattern. Skills with suffixes `-fix`, `-sync`, `-prune`, `-rebuild` (verbs that imply action) mutate by default; in this package, `/all-fix` and `/grug-fix`. Bare verbs `/commit` and `/pause` are session-scoped exceptions.

## Classification Taxonomy

Every user-invocable skill is one of:

- **Mutating skill** (default behavior). May carry a `--report` flag when preview is useful. Some mutating skills have no `--report` because preview is meaningless (the operator runs `git diff` first) or the action is small and reversible.
- **Pure report** (never mutates). No `--report` flag because there is nothing to invert.

A skill is classified by its actual behavior, not its name. The Arguments table lead-in states the classification in one sentence so a reader knows the default at a glance.

## Required Section Structure

Each user-invocable skill that takes arguments uses these headings in this order, omitting any that do not apply:

1. `## Arguments` (required when the skill takes arguments). Contains the table and a lead-in sentence stating the classification.
2. `## Scope` (optional). Include only when scope vocabulary needs detection order, fallback behavior, or base-branch resolution beyond what the Arguments table can express.
3. `## Mutation` (optional). Include only when mutation behavior needs clarification beyond a single table row.

`## Customization` is retired as a section name. New skills do not introduce it.

## Worked Examples

### Mutating with `--report`: `/all-fix`

`/all-fix` runs the default code-lens reviews in parallel, prints the aggregated report, then applies non-conflicting findings. `--report` skips only the apply phase. Its Arguments table:

| Input | Effect |
|-------|--------|
| (no argument) | Apply fixes from default four lenses to changed files (staged + unstaged) |
| `all` | Apply fixes from default four lenses across the full codebase (sampled for high-risk and high-traffic modules) |
| `<path>` `<glob>` | Apply fixes from default four lenses scoped to the path or pattern |
| `<lens-names>` | Run only the named lenses from `grug`, `honest-code`, `tidy-first`, `parse-dont-validate` |
| `+aposd` | Add the APOSD lens to the default set (documented exemption) |
| `+legacy-code` | Add the Legacy Code lens to the default set (documented exemption) |
| `--report` | Aggregate findings and print the report; skip the apply phase |

Lead-in: "Interpret naturally. This skill mutates by default. Pass `--report` to aggregate findings without writing."

### Pure report: `/grug-review`

`/grug-review` reviews code through grug brain philosophy and prints findings. Never mutates. Its Arguments table:

| Input | Action |
|-------|--------|
| (no argument) | Review changed files only (staged + unstaged) |
| `all` | Review full codebase (sample high-risk and high-traffic modules) |
| `<path>` `<glob>` | Review files under the path or matching the pattern |

Lead-in: "Interpret naturally. This skill is a pure report. It does not mutate the workspace and carries no `--report` flag (there is nothing to invert)."

### Problem-input exemption: `/grug-fix`

`/grug-fix` operates on a bug, not a code scope. The canonical scope rows do not apply. Its Arguments table carries problem-entry-point rows and a `--report` row:

| Input | Action |
|-------|--------|
| (no argument) | Ask what is broken |
| error message or stack trace | Start from the error |
| `path/to/file.ts:42` | Start from a specific location |
| failing test name or path | Start from the test failure |
| description of unexpected behavior | Start from the symptom |
| `--report` | Diagnose and propose the fix; do not edit files |

Lead-in: "Interpret naturally. This skill mutates by default. Pass `--report` to diagnose and propose the fix without editing files."

## Exemptions

Two exemptions from the canonical grammar are recognized in this plugin:

### Additive lens sigils in `/all-fix`

`/all-fix` uses `+aposd` and `+legacy-code` to add opt-in lenses to its default set. Bare names already serve a different role in `/all-fix`: a bare lens name selects a subset of the defaults (`/all-fix grug honest-code` runs only those two). The `+` sigil disambiguates additive from subset semantics in a way bare phrases cannot express compactly. The sigil is permitted only in this skill.

### Problem-input grammar in `/grug-fix`

`/grug-fix` operates on a bug, not a code scope. Its Arguments table replaces the canonical scope rows with problem-entry-point rows (error message, file:line, failing test name, behavior description). The `all` keyword is not accepted because "debug the whole codebase" has no useful meaning. Other mutating skills that operate on a problem rather than a region of code may follow this pattern; document the exemption in the skill body when they do.

## Ambiguity Notes

- **`(no argument)` outside a git worktree.** Skills whose narrowest default is `git diff` ask the operator what to operate on when no diff exists. They do not fall through silently to `all`.
- **`--report` versus natural-language synonyms.** Only the literal token `--report` enables report-only mode. Phrases like "preview", "dry run", or "just tell me" are scope input or noise, not mode triggers. This keeps the mode boundary unambiguous.
- **`commit` mixing committed and staged state.** When a skill accepts the `commit` opt-in row, it reads the most recent commit message and the currently staged draft together. The skill body states the precedence rule when both contribute.

## Author Checklist

When adding or modifying a user-invocable skill:

- Frontmatter `name` matches the skill's directory name.
- `## Arguments` heading is present.
- File-aware skills include the three core scope rows (`(no argument)`, `all`, `<path>` `<glob>`).
- Opt-in rows (`#N`, `pr`, `commit`) appear only when the skill genuinely supports them.
- Mutating skill: default applies changes; a `--report` row is present unless preview is meaningless or the action is small and reversible (state the reason in the lead-in).
- Pure report: no `--report` flag; the lead-in states "carries no `--report` flag (there is nothing to invert)".
- Mirror at `.opencode/skills/<name>/SKILL.md` is byte-identical to `.apm/skills/<name>/SKILL.md` (verify with `diff -q`).
- `opencode.jsonc` `permission.skill` lists the skill with `"allow"`.
- `agents/openai.yaml` carries `interface.display_name`, `short_description`, `default_prompt`, and `policy.allow_implicit_invocation`.
- Cross-references in `README.md` and `CHANGELOG.md` are updated.
- Any exemption from the canonical grammar is documented in this file's Exemptions section.
