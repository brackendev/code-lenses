# Code Lenses for Claude Code

Code Lenses provides Claude Code with a set of reusable review, debugging, and implementation-guidance commands built around grug brain, Honest Code, Tidy First?, and A Philosophy of Software Design.

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

Runs all four code lens reviews in parallel using the packaged Claude reviewer agents, then aggregates the findings.

```bash
/code-lenses:review-all
/code-lenses:review-all src/api/
/code-lenses:review-all all
/code-lenses:review-all grug aposd
/code-lenses:review-all tidy-first src/services/
```

### `/code-lenses:honest-code-review [scope or options...]`

Reviews code for dishonest patterns using the 11 Honest Code constructs.

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

### Review and debug commands

| Skill | Purpose |
|-------|---------|
| `review-all` | Run all four review lenses and aggregate the findings |
| `grug-review` | Review for complexity, over-engineering, and abstraction debt |
| `aposd-review` | Review for module depth, information hiding, and complexity symptoms |
| `honest-code-review` | Review for dishonest patterns using the Honest Code constructs |
| `tidy-first-review` | Review for tidying opportunities and mixed structural and behavioral changes |
| `grug-debug` | Debug through small repro, evidence, and one-change-at-a-time workflow |

### Auto-triggered implementation skills

These skills apply automatically based on conversation context and are not invoked directly as slash commands.

| Skill | Triggers |
|-------|----------|
| `grug` | Simplicity, over-complexity, abstraction, architecture, refactoring, test design |
| `honest-code` | Declarative design, pure functions, flat data, no classes, dishonest patterns |
| `tidy-first` | Tidying before a change, guard clauses, reading order, dead code, structural prep |
| `aposd` | Deep modules, information hiding, information leakage, interface depth, complexity symptoms |
