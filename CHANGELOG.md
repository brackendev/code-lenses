# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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
