# Code Lenses for Claude Code

Install, update, and use the `code-lenses` package in Claude Code.

## Install

### With APM

Per-project:

```bash
apm install --target claude brackendev/code-lenses/claude/code-lenses
```

Global:

```bash
apm install -g --target claude brackendev/code-lenses/claude/code-lenses
```

APM deploys these skills into `.claude/skills/`.

Remove it with:

```bash
apm uninstall --target claude brackendev/code-lenses/claude/code-lenses
```

Remove a global install with:

```bash
apm uninstall -g --target claude brackendev/code-lenses/claude/code-lenses
```

### With Claude Marketplace

```bash
claude plugins marketplace add brackendev/code-lenses
claude plugins install code-lenses@code-lenses
```

Remove them with:

```bash
claude plugins uninstall code-lenses
```

## Update

### APM Installs

Update all project-scoped installs from the project root:

```bash
apm deps update --target claude
```

Update one package:

```bash
apm deps update --target claude brackendev/code-lenses/claude/code-lenses
```

Update global installs:

```bash
apm deps update -g --target claude brackendev/code-lenses/claude/code-lenses
```

### Claude Marketplace

Refresh marketplace metadata:

```bash
claude plugins marketplace update code-lenses
```

Update installed plugins:

```bash
claude plugins update code-lenses
```

If the plugin was installed outside the default user scope, pass the matching scope to the update command, for example `claude plugins update -s project code-lenses`.

## Use

```text
/code-lenses:review-all
/code-lenses:grug-review
/code-lenses:grug-debug TypeError: Cannot read property
/code-lenses:grug
```
