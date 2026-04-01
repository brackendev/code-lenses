# Code Lenses for Codex

Code Lenses provides Codex with a set of reusable review, debugging, and implementation-guidance skills built around grug brain, Honest Code, Tidy First?, A Philosophy of Software Design, Parse Don't Validate, and Working Effectively with Legacy Code.

This package lives in `./codex/code-lenses`.

## What It Provides

Code Lenses helps Codex:

- review the current diff through multiple design lenses
- debug through a minimal reproduction and evidence-first workflow
- bias implementation work toward simpler, cleaner code decisions

This package is used through skills referenced in prompts in Codex.

## Install

Install the Codex skills from GitHub with the built-in `skill-installer` skill:

```text
$skill-installer https://github.com/brackendev/code-lenses
```

Restart Codex to pick up new skills.

The Codex package in this repo lives at `./codex/code-lenses`, and the bundled skills live under `./codex/code-lenses/skills/`.

## How To Use It

After the skills are installed, use the skill names directly in your prompt when you want Codex to apply a lens.

You can also type `@` to select one of the installed skills explicitly.

Examples:

```text
Use review-all on the current diff.
Use grug-review on src/auth/.
Use aposd-review on this module boundary change.
Use tidy-first-review on these refactor edits.
Use honest-code-review on this state management code.
Use parse-dont-validate-review on the input handling code.
Use legacy-code-review on this untested module.
Use grug-debug on this failing test.
Use review-all +legacy-code on the current diff.
```

## Bundled Lenses

See the [root README](../README.md#bundled-lenses) for the full list of review, debug, and auto-applied implementation skills.
