# Code Lenses for Codex

Install the `code-lenses` package in Codex via APM.

> **Global installs:** APM 0.8.11 warns that Codex has no native user-scope deployment. The `-g` flag is shown below for completeness but is not reliable today. Prefer per-project installs.

APM deploys skills into `.agents/skills/`.

## Install

Per-project:

```bash
apm install --target codex brackendev/code-lenses/codex/code-lenses
```

Global: add `-g`.

## Update

```bash
apm deps update --target codex                                                # every project install
apm deps update --target codex brackendev/code-lenses/codex/code-lenses       # this package
apm deps update -g --target codex brackendev/code-lenses/codex/code-lenses    # global install
```

## Uninstall

Add `-g` for global:

```bash
apm uninstall brackendev/code-lenses/codex/code-lenses
```

## Use

Codex skills are model-invoked. Phrase requests in natural language:

```text
Use review-all on the current diff.
Use grug-review on src/auth/.
Use grug-debug on this failing test.
Use grug for this refactor.
```
