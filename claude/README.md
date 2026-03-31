# Code Lenses for Claude Code

Code Lenses provides Claude Code with a set of reusable review, debugging, and implementation-guidance commands built around grug brain, Honest Code, Tidy First?, A Philosophy of Software Design, Parse Don't Validate, and Working Effectively with Legacy Code.

This package lives in `./claude/code-lenses`.

## What It Provides

Code Lenses helps Claude Code:

- review the current diff through multiple design lenses
- debug through a minimal reproduction and evidence-first workflow
- bias implementation work toward simpler, cleaner code decisions

This package is installed and invoked as a Claude plugin with slash commands.

## Install

This repo includes the Claude package and marketplace metadata:

| Path | Purpose |
|------|---------|
| `./claude/code-lenses` | Claude package root |
| `./claude/code-lenses/.claude-plugin/plugin.json` | Claude plugin manifest |
| `./.claude-plugin/marketplace.json` | Repo-level Claude marketplace entry |

Add the marketplace:

```bash
/plugin marketplace add brackendev/code-lenses
```

Install the plugin:

```bash
/plugin install code-lenses@code-lenses
```

Uninstall:

```bash
/plugin uninstall code-lenses@code-lenses
/plugin marketplace remove brackendev/code-lenses
```

## How To Use It

Invoke Code Lenses through slash commands.

### `/code-lenses:review-all [scope or options...]`

Runs five code lens reviews in parallel by default, then aggregates the findings. Legacy Code is opt-in.

```bash
# Default 5 lenses on changed files
/code-lenses:review-all

# Default 5 lenses on a specific path or the full codebase
/code-lenses:review-all src/api/
/code-lenses:review-all all

# Pick specific lenses
/code-lenses:review-all grug aposd
/code-lenses:review-all tidy-first src/services/

# Include Legacy Code (opt-in, not in the default set)
/code-lenses:review-all +legacy-code
/code-lenses:review-all +legacy-code src/services/
```

### `/code-lenses:parse-dont-validate-review [scope or options...]`

Reviews code for type-driven correctness using Parse Don't Validate and Make Illegal States Unrepresentable.

```bash
/code-lenses:parse-dont-validate-review
/code-lenses:parse-dont-validate-review src/models/
/code-lenses:parse-dont-validate-review all
```

### `/code-lenses:legacy-code-review [scope or options...]`

Reviews code for safe modification opportunities using Working Effectively with Legacy Code techniques.

```bash
/code-lenses:legacy-code-review
/code-lenses:legacy-code-review src/services/
/code-lenses:legacy-code-review all
```

### `/code-lenses:honest-code-review [scope or options...]`

Reviews code for dishonest patterns using the Honest Code constructs.

```bash
/code-lenses:honest-code-review
/code-lenses:honest-code-review src/models/
/code-lenses:honest-code-review all
```

### `/code-lenses:grug-review [scope or options...]`

Reviews changed code for complexity demons through grug brain philosophy.

```bash
/code-lenses:grug-review
/code-lenses:grug-review src/auth/
/code-lenses:grug-review all
```

### `/code-lenses:tidy-first-review [scope or options...]`

Reviews code for tidying opportunities using Tidy First? and detects mixed structural and behavioral changes.

```bash
/code-lenses:tidy-first-review
/code-lenses:tidy-first-review src/services/
/code-lenses:tidy-first-review all
```

### `/code-lenses:aposd-review [scope or options...]`

Reviews code for module depth, information hiding, and complexity using A Philosophy of Software Design.

```bash
/code-lenses:aposd-review
/code-lenses:aposd-review src/services/
/code-lenses:aposd-review all
```

### `/code-lenses:grug-debug [bug description, error, or failing test...]`

Debugs through reproduce, shrink, inspect, verify, and prove.

```bash
/code-lenses:grug-debug
/code-lenses:grug-debug TypeError: Cannot read property
/code-lenses:grug-debug src/auth.ts:42
/code-lenses:grug-debug test_login_redirect
```

## Bundled Lenses

See the [root README](../README.md#bundled-lenses) for the full list of review, debug, and auto-triggered implementation skills.
