# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.20] - 2026-06-15

### Changed

- Add Kiro to the README's runtime list. APM 0.20.0 added Kiro as a first-class install target included in `apm install --target all`, so the README now lists it alongside Claude Code, Codex, OpenCode, Cursor, Copilot, Gemini, and Windsurf.

## [0.1.19] - 2026-05-28

### Added

- `CONVENTIONS.md` gains Rule 4: vendored and generated paths are excluded by default from mutating skills that walk the workspace. Two filters apply together (`.gitignore` matches plus a hardcoded floor of dependency directories, build outputs, and lock files). The override rides on Rule 1's existing `<path>` `<glob>` grammar; no new flag is introduced. The Author Checklist gains a matching item. The pure-report review skills are advisory under Rule 4 because reading is not modification.
- `/lenses-fix` gains a `## Scope` section that restates Rule 4 in context. `/grug-fix` gains a `## Scope` section that describes how the filter applies when a bug entry point resolves into a vendored path: the skill diagnoses but does not write unless the operator explicitly names the vendored file.

## [0.1.18] - 2026-05-26

### Fixed

- Quote the YAML `description` frontmatter in the six review skills that were not already quoted. Prevents potential YAML misinterpretation of special characters in description values.

## [0.1.17] - 2026-05-26

### Fixed

- Quote the YAML `argument-hint` frontmatter in all eight skills. Unquoted square brackets were parsed as YAML flow sequences, which caused `lenses-fix` and `grug-fix` to fail with "did not find expected key" errors when the value contained multiple bracket groups. The remaining six review skills had single bracket groups that parsed as arrays instead of strings rather than failing outright.

## [0.1.16] - 2026-05-26

### Fixed

- Quote the YAML `description` frontmatter in `grug-fix` and `lenses-fix` so the `--` sequences are not misinterpreted as YAML block indicators. The unquoted values caused YAML parse errors that prevented both skills from loading.

## [0.1.15] - 2026-05-21

### Changed

- The `all-fix` skill is renamed to `lenses-fix` so the noun half of the canonical `<target>-<verb>` pattern names what the skill actually targets (the code-design lenses), not an inaccurate scope. The previous name suggested a meta-runner over every fix skill in the family; the skill in fact runs the default code lenses in parallel and applies their non-conflicting findings. Operators with a saved `/all-fix` invocation should replace it with `/lenses-fix`. The skill's behavior, default lens set, additive `+aposd` and `+legacy-code` sigils, and `--report` flag are unchanged; only the name moves.

## [0.1.14] - 2026-05-20

### Changed

- `CONVENTIONS.md` Rule 3 now states the mutation default with noun-first canonical phrasing (`Skills with suffixes -fix, -sync, -prune, -rebuild ... mutate by default`) instead of the legacy verb-first prefix patterns (`/fix-*`, `/sync-*`, `/prune-*`, `/rebuild-*`). The change keeps the rule aligned with the rest of the agent-skills family after the noun-first rename.
- `CONTRIBUTING.md` is aligned to the family-wide structural template (Layout / APM lockfile rule / Adding or modifying a skill / Validation / Skill conventions). A `CONVENTIONS.md` row is added to the Layout table, a version-bump step is added to the skill-modification procedure, and a new Skill conventions section documents the user-invocable / model-invocable distinction with package-specific example skills.

## [0.1.13] - 2026-05-20

### Changed

- The `fix-all` skill is renamed to `all-fix` to adopt the noun-first canonical naming pattern (`<target>-<verb>`) shared across the agent-skills family. The verb suffix `-fix` consistently signals a mutating quality pipeline. Operators with a saved `/fix-all` invocation should replace it with `/all-fix`. The skill's behavior, default lens set, additive `+aposd` and `+legacy-code` sigils, and `--report` flag are unchanged; only the name moves.

## [0.1.12]

### Added

- `CONVENTIONS.md` at the repository root documenting the canonical argument grammar, scope vocabulary, and mutation default for every user-invocable skill. Three rules: skills accept natural-language keywords and bare paths with `--report` as the only sanctioned flag; file-aware skills share a three-row scope vocabulary (`(no argument)`, `all`, `<path>` `<glob>`); skills that can mutate the workspace apply changes by default and accept `--report` to preview. Documents the `+aposd` / `+legacy-code` additive sigil and the problem-input grammar as recognized exemptions.
- `--report` flag on `/grug-fix` and `/fix-all` to produce the findings or proposed fix without modifying any files.

### Changed

- Renamed `/grug-debug` to `/grug-fix`. The skill applies the fix and adds a regression test by default; pass `--report` to diagnose and propose the fix without editing files. Update saved invocations from `/grug-debug` to `/grug-fix`.
- Renamed `/review-all` to `/fix-all`. The skill aggregates findings from the default code lenses in parallel, prints a unified report, then applies non-conflicting findings by default. Pass `--report` to print the aggregated report and skip the apply phase. Update saved invocations from `/review-all` to `/fix-all`.
- The user-invocable review skills (`/aposd-review`, `/grug-review`, `/honest-code-review`, `/legacy-code-review`, `/parse-dont-validate-review`, `/tidy-first-review`) now present their argument table under a single `## Arguments` heading with three canonical scope rows: `(no argument)`, `all`, and `<path>` `<glob>`. Behavior is unchanged.

### Removed

- The `fix` modifier on `/review-all` (now `/fix-all`) is removed. Mutation is now the default; pass `--report` to opt out of applying fixes. Update saved invocations from `/review-all <scope> fix` to `/fix-all <scope>`.

## [0.1.11]

### Fixed

- Remove Claude Code-specific tool name from `grug-review`, `honest-code-review`, `tidy-first-review`, `parse-dont-validate-review`, `aposd-review`, and `legacy-code-review`. The skill bodies told the host to "Use Bash tool for scope discovery," which named a tool that only exists on Claude Code. The instruction is now runtime-neutral and reads "Run these commands for scope discovery."
- `review-all`: Remove the partial runtime example list from the sub-agent launch instruction. The skill now relies on the existing "whatever sub-agent mechanism the host runtime provides" wording without naming a subset of runtimes.

## [0.1.10]

### Fixed

- `review-all`: Remove Codex-specific wording from the skill body. The previous text instructed every host to "launch sub-agents using Codex," which caused non-Codex runtimes (Claude Code, OpenCode, Gemini, Cursor, Copilot, Windsurf) to call the Codex MCP server instead of their own sub-agent mechanism. The skill is now runtime-neutral and uses whatever sub-agent mechanism the host runtime provides.

## [0.1.9]

### Changed

- Repackaged as a single APM `type: skill` plugin. Install with `apm install brackendev/code-lenses --target all` at project scope or add `-g` for user scope. The legacy `brackendev/code-lenses/claude/code-lenses` and `brackendev/code-lenses/codex/code-lenses` install paths are removed.
- One install now deploys 13 skills to every runtime APM supports (Claude Code, Codex, OpenCode, Cursor, Copilot, Gemini, Windsurf) instead of separate per-runtime packages.
- `review-all` runs the lens skills directly as parallel sub-agents instead of dispatching to named reviewer subagents.

### Removed

- Removed the per-runtime reviewer subagents (`grug-reviewer`, `honest-code-reviewer`, `tidy-first-reviewer`, `parse-dont-validate-reviewer`, `aposd-reviewer`, `legacy-code-reviewer`). The review skills now act directly.
- Removed the Claude marketplace (`.claude-plugin/marketplace.json`) and Codex marketplace (`.agents/plugins/marketplace.json`) entries. The plugin installs through APM only.

## [0.1.8] - 2026-04-13

### Removed

- Remove `effort` frontmatter from all skills (both Claude and Codex packages)

## [0.1.7] - 2026-04-12

### Changed

- All review skills (`grug-review`, `honest-code-review`, `tidy-first-review`, `aposd-review`, `parse-dont-validate-review`, `legacy-code-review`, `review-all`): Increase effort level from `high` to `max` for deeper analysis.
- **Packaging:** Added an APM-detectable root `plugin.json` to the Codex package so `apm install --target codex brackendev/code-lenses/codex/code-lenses` works for per-project installs.

## [0.1.6] - 2026-04-06

### Added

- `review-all`: Add `fix` modifier to apply non-conflicting findings after the review. Conflicting advice (where lenses disagree) is skipped and reported.

## [0.1.5] - 2026-04-03

### Changed

- `grug`, `honest-code`, `tidy-first`, `parse-dont-validate`, `aposd`: Revert to auto-triggered (description-based pattern matching) instead of user-invocable. The agent activates these skills automatically when conversation context matches trigger patterns in the skill description.

## [0.1.4] - 2026-04-01

### Changed

- `review-all`: Reduce default lenses from five to four. Move APOSD to opt-in alongside Legacy Code to reduce conflicting advice during reviews.
- `aposd`: Change to explicit implementation skill. Invoke with `/code-lenses:aposd` (Claude) or `Use aposd` (Codex).
- `grug`, `honest-code`, `tidy-first`, `parse-dont-validate`: Make user-invocable. The agent may still apply them automatically when relevant, but explicit invocation is now guaranteed.
- Add "Best for" column to all skill tables in the root README with concrete scenarios and tech stack examples.

## [0.1.3] - 2026-03-31

### Changed

- `parse-dont-validate`: Add Clojure row to language-specific techniques table (`clojure.spec`/Malli, smart constructors, namespaced keys, tagged maps)

## [0.1.2] - 2026-03-31

### Added

- `parse-dont-validate`: Auto-triggered skill applying Parse Don't Validate and Make Illegal States Unrepresentable principles to code changes
- `parse-dont-validate-review`: Review code for type-driven correctness, boundary parsing, and illegal states with severity tiers UNGUARDED, LEAKING, LOOSE
- `legacy-code-review`: Review code for safe modification using Working Effectively with Legacy Code techniques with severity tiers UNTESTED, BRITTLE, RIGID
- `parse-dont-validate-reviewer`, `legacy-code-reviewer` agents for parallel review execution
- `honest-code` Construct 12: Push Effects to the Edges, based on Gary Bernhardt's Functional Core, Imperative Shell pattern

### Changed

- `honest-code`: Add Construct 12 (Push Effects to the Edges, from Gary Bernhardt) alongside the 11 original constructs from honestcode.software
- `honest-code-review`: Updated to evaluate code against all constructs including Construct 12
- `review-all`: Expanded from four to five default parallel review lenses (adding Parse Don't Validate), with Legacy Code available as opt-in

## [0.1.1] - 2026-03-31

### Changed

- `review-all`: Add Conflicts section to aggregated report, surfacing contradictory advice between lenses instead of silently dropping it

## [0.1.0] - 2026-03-26

### Added

- `review-all`: Run all four code lens reviews in parallel using specialized agents with aggregated report
- `grug`: Auto-triggered skill applying grug brain developer philosophy to code changes
- `grug-debug`: Debug problems through grug brain philosophy with structured investigation process
- `grug-review`: Review code for complexity demons with severity tiers CLUB, CONCERN, GRUMBLE
- `honest-code`: Auto-triggered skill applying the 11 Honest Code constructs from honestcode.software
- `honest-code-review`: Review code for dishonest patterns with severity tiers CRIME SCENE, SUSPECT, WITNESS
- `tidy-first`: Auto-triggered skill applying Tidy First? philosophy for separating structural and behavioral changes
- `tidy-first-review`: Review code for tidying opportunities with severity tiers TANGLED, CLUTTERED, DUSTY
- `aposd`: Auto-triggered skill applying A Philosophy of Software Design principles for deep modules and information hiding
- `aposd-review`: Review code for module depth, information hiding, and complexity symptoms with severity tiers SHALLOW, EXPOSED, SURFACE
- `grug-reviewer`, `aposd-reviewer`, `honest-code-reviewer`, `tidy-first-reviewer` agents for parallel review execution
