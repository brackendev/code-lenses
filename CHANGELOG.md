# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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
