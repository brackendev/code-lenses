# Code Lenses for Codex

Install, update, and use the `code-lenses` package in Codex.

## Install

Per-project:

```bash
apm install --target codex brackendev/code-lenses/codex/code-lenses
```

Global:

```bash
apm install -g --target codex brackendev/code-lenses/codex/code-lenses
```

Per-project installs deploy these skills into `.agents/skills/`.

APM 0.8.11 currently warns that Codex does not have native user-scope deployment support, so the global commands above are not reliable today. Prefer per-project Codex installs.

## Update

Update all project-scoped installs from the project root:

```bash
apm deps update --target codex
```

Update one package:

```bash
apm deps update --target codex brackendev/code-lenses/codex/code-lenses
```

Update global installs:

```bash
apm deps update -g --target codex brackendev/code-lenses/codex/code-lenses
```

Codex global updates have the same current limitation as Codex global installs.

## Use

```text
Use review-all on the current diff.
Use grug-review on src/auth/.
Use grug-debug on this failing test.
Use grug for this refactor.
```
