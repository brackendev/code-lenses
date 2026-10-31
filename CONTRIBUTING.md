# Contributing

For people working on the plugin source. End-user install instructions live in [README.md](README.md).

## Layout

| Path | Purpose |
|------|---------|
| `apm.yml` | Package manifest (`type: skill`, no MCP dependencies). |
| `.apm/skills/<name>/` | Canonical skill source. APM deploys from here. Each directory holds `SKILL.md` plus optional `agents/openai.yaml`, `references/`, `scripts/`, `examples/`. |
| `.opencode/skills` | Symlink to `../.apm/skills`. OpenCode finds it from any directory in the repository, so an OpenCode session loads the canonical skills and their supporting files without an APM install. |
| `opencode.jsonc` | Local OpenCode configuration. |
| `README.md` | End-user documentation. |
| `CHANGELOG.md` | User-facing changes per version. |
| `CONVENTIONS.md` | Canonical argument grammar, scope vocabulary, and mutation defaults for every user-invocable skill. |
| `AGENTS.md`, `GEMINI.md`, `TODO.md` | Local agent instructions (`GEMINI.md` is a symlink to `AGENTS.md`) and working notes. Gitignored globally and never committed. |

## APM lockfile rule

Do not add `.claude-plugin/`, `.codex-plugin/`, `.agents/plugins/marketplace.json`, or a root `plugin.json`. Any of those change the APM lockfile classification from `apm_package` to `marketplace_plugin` and suppress skill deployment to every runtime.

## Adding or modifying a skill

1. Edit `.apm/skills/<name>/SKILL.md`.
2. Update `README.md` if the change is user-facing.
3. Add a `CHANGELOG.md` entry for every change to a published file, including documentation-only changes, under a `## [x.y.z] - YYYY-MM-DD` heading for the new `apm.yml` version. Leave `## [Unreleased]` empty above it.
4. Increment the `version` field in `apm.yml`.

## Vendored sources

A lens whose source publishes a canonical rule file ships that file unchanged instead of a summary. `.apm/skills/honest-code/references/honest-code-principles.md` is the [Honest Code Principles](https://github.com/openhonest/honest-code-principles) file at a pinned commit, under the Apache License 2.0. The upstream README asks projects to depend on its repository and neither to paste the text into a project nor to summarize it, because copies drift. This package ships an exact copy anyway, because a skill has to work in sessions without network access. To keep the copy from drifting, it is pinned to one commit and compared with upstream before each release (step 3 below), and the skills never summarize it. They cite principles by number (`P15`), and their examples are short pointers to the cited principle, not definitions.

To update the copy:

1. Read the upstream commits since the pinned one. A new or renumbered principle can change the examples in `honest-code-review/SKILL.md` and the principle numbers cited in the other lenses and in `lenses-fix/references/stack-adaptation.md`.
2. Replace everything below the header comment with the upstream file, and update the URL, commit, and date in the header.
3. Confirm the body matches upstream exactly: `tail -n +9 .apm/skills/honest-code/references/honest-code-principles.md | diff - <(gh api "repos/openhonest/honest-code-principles/contents/honest-code-principles.md?ref=<commit>" --jq .content | base64 -d)`.
4. Search for cited principle numbers (`grep -rn "P[0-9][0-9]" .apm`) and update any that changed.

Rules that this package adds to a lens, such as the severity tiers in every review skill, are labeled as this package's own and never attributed to the source.

## Validation

Configuration files:

```bash
python3 -m json.tool opencode.jsonc >/dev/null
python3 -c "import yaml, pathlib; yaml.safe_load(pathlib.Path('apm.yml').read_text())"
python3 -c "import yaml, pathlib; [yaml.safe_load(p.read_text()) for p in pathlib.Path('.apm/skills').glob('*/agents/openai.yaml')]"
```

Runtime install (requires `apm` and the runtime CLIs you want to verify: `claude`, `codex`, `copilot`, `gemini`, `opencode`):

- Run `apm install --target all`, `apm update`, and `apm uninstall` in a clean temporary project. Pre-create the runtime roots: `.agents/`, `.claude/`, `.cursor/`, `.opencode/`, `.gemini/`, `.github/`, `.windsurf/`. With several runtime roots present, APM stops with "Multiple harnesses detected" unless the install names a target. `apm update` reuses the targets that the install records in the project's `apm.yml`.
- Exercise user-scope with `apm install brackendev/code-lenses -g [--target ...]` and `apm uninstall brackendev/code-lenses -g`. A local filesystem path (`apm install /absolute/path -g`) is also accepted.
- Confirm OpenCode deployment from the temporary project with `opencode api GET "/api/skill?location[directory]=$PWD"`, which lists the skills OpenCode discovers and their source paths. The first request for a directory can return an empty list while OpenCode loads the skills, so repeat it before reading the result.

## Skill conventions

For the argument grammar, scope vocabulary, and mutation defaults that every user-invocable skill follows, see [CONVENTIONS.md](CONVENTIONS.md).

| Setting | When to use |
|---------|-------------|
| `user-invocable: true`, `disable-model-invocation: true` | User-only slash command, for a skill whose changes reach beyond the working tree or are hard to undo. |
| `user-invocable: true`, no `disable-model-invocation` | Slash command the model can also run from a natural-language request (for example `grug-review`, `lenses-fix`). Write the description as a trigger. |
| `user-invocable: false` | Model-invoked from conversation context (for example `grug`, `honest-code`). |

Every skill carries `agents/openai.yaml` whose `policy.allow_implicit_invocation` matches the table above: `false` for a user-only skill and `true` for every skill the model can run.
