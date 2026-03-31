# Code Lenses for Codex

Code Lenses provides Codex with a set of reusable review, debugging, and implementation-guidance skills built around grug brain, Honest Code, Tidy First?, A Philosophy of Software Design, Parse Don't Validate, and Working Effectively with Legacy Code.

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

To install this plugin in Codex:

1. Open the repository root in Codex, not the `./codex/` subdirectory. Codex needs the repo root so it can see `./.agents/plugins/marketplace.json`.
2. Restart Codex if this repo was already open before the marketplace file or plugin files were added or changed.
3. Open the plugin directory:
   - In the Codex app, open `Plugins`.
   - In Codex CLI, run `codex` and enter `/plugins`.
4. Find the repo marketplace entry and install `Code Lenses`.
5. Start a new thread and ask Codex to use one of the bundled skills.

The marketplace entry points Codex to `./codex/code-lenses`, and the bundled skills live under `./codex/code-lenses/skills/`.

## How To Use It

After the plugin is installed, use the skill names directly in your prompt when you want Codex to apply a lens.

You can also type `@` to select the plugin or one of its bundled skills explicitly.

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
