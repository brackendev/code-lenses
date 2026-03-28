# Code Lenses for Codex

Code Lenses provides Codex with a set of reusable review, debugging, and implementation-guidance skills built around grug brain, Honest Code, Tidy First?, and A Philosophy of Software Design.

This package lives in `./codex/code-lenses`.

## What It Provides

Code Lenses helps Codex:

- review the current diff through multiple design lenses
- debug through a minimal reproduction and evidence-first workflow
- bias implementation work toward simpler, cleaner code decisions

This package is used through skills referenced in prompts in Codex.

## Setup

This repo includes repo-scoped marketplace metadata for Codex:

| Path | Purpose |
|------|---------|
| `./codex/code-lenses` | Codex package root |
| `./codex/code-lenses/.codex-plugin/plugin.json` | Codex plugin manifest |
| `./.agents/plugins/marketplace.json` | Repo-level Codex marketplace entry |

Open this repository in Codex with the repo-scoped marketplace metadata available. The marketplace entry points Codex to `./codex/code-lenses`, and the bundled skills live under `./codex/code-lenses/skills/`.

## How To Use It

Use the skill names directly in your prompt when you want Codex to apply a lens.

Examples:

```text
Use review-all on the current diff.
Use grug-review on src/auth/.
Use aposd-review on this module boundary change.
Use tidy-first-review on these refactor edits.
Use honest-code-review on this state management code.
Use grug-debug on this failing test.
```

## Bundled Lenses

### Review and debug skills

| Skill | Purpose |
|-------|---------|
| `review-all` | Run all four review lenses and aggregate the findings |
| `grug-review` | Complexity and over-engineering review |
| `aposd-review` | Module depth and information-hiding review |
| `honest-code-review` | Honest Code construct review |
| `tidy-first-review` | Tidying and mixed-change review |
| `grug-debug` | Evidence-first debugging workflow |

### Auto-applied implementation skills

| Skill | Purpose |
|-------|---------|
| `grug` | Simplicity and anti-complexity guidance during implementation |
| `aposd` | Deep module and information-hiding guidance |
| `honest-code` | Honest constructs guidance during implementation |
| `tidy-first` | Structural tidying guidance before behavioral change |
