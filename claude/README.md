# Code Lenses for Claude Code

Install the `code-lenses` package in Claude Code via APM or the native Claude marketplace.

## APM

APM deploys skills into `.claude/skills/`.

Install (per-project):

```bash
apm install --target claude brackendev/code-lenses/claude/code-lenses
```

Install (global): add `-g`.

Update:

```bash
apm deps update --target claude                                                # every project install
apm deps update --target claude brackendev/code-lenses/claude/code-lenses      # this package
apm deps update -g --target claude brackendev/code-lenses/claude/code-lenses   # global install
```

Uninstall (add `-g` for global):

```bash
apm uninstall brackendev/code-lenses/claude/code-lenses
```

## Claude Marketplace

Add the marketplace once:

```bash
claude plugins marketplace add brackendev/code-lenses
```

Install:

```bash
claude plugins install code-lenses@code-lenses
```

Update:

```bash
claude plugins marketplace update code-lenses   # refresh marketplace metadata
claude plugins update code-lenses
```

If the plugin was installed outside the default user scope, pass `-s <scope>` to the update command (for example `-s project`).

Uninstall:

```bash
claude plugins uninstall code-lenses
```

## Use

```text
/code-lenses:review-all
/code-lenses:grug-review
/code-lenses:grug-debug TypeError: Cannot read property
/code-lenses:grug
```
