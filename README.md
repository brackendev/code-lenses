# code-lenses

Software design lenses for code review and implementation guidance, packaged as an [APM](https://github.com/microsoft/apm) plugin. One install deploys 13 skills (grug brain, Honest Code, Tidy First?, A Philosophy of Software Design, Parse Don't Validate, Legacy Code) to every runtime APM supports: Claude Code, Codex, OpenCode, Cursor, Copilot, Gemini, and Windsurf.

Skills follow the [Agent Skills](https://agentskills.io) open standard. Review and debug skills appear as slash commands (`/review-all`, `/grug-review`, and so on); the five implementation-guidance skills activate automatically from conversation context.

Source: <https://github.com/brackendev/code-lenses>. APM shorthand: `brackendev/code-lenses`.

## Install

This plugin is distributed through APM, so install [APM](https://github.com/microsoft/apm) first if you don't already have it. Then, in a project:

```bash
apm install brackendev/code-lenses --target all
```

Globally for your user account:

```bash
apm install brackendev/code-lenses -g --target all
```

Update with `apm update [-g]`. Remove with `apm uninstall brackendev/code-lenses [-g]`. A local filesystem path can replace the shorthand at either scope.

## Quick start

Slash commands run inside your agent runtime (Claude Code, Codex CLI, OpenCode, and the rest), not at a shell prompt. The shell-styled code blocks below are formatted that way for readability.

Run every lens in parallel against the current diff and aggregate the findings:

```bash
/review-all
```

Run a single lens against changed files:

```bash
/grug-review
/honest-code-review
/tidy-first-review
/parse-dont-validate-review
```

Add an opt-in lens to the default set:

```bash
/review-all +aposd
/review-all +legacy-code
```

Apply non-conflicting fixes after a review:

```bash
/review-all fix
```

Debug a failing test through grug brain philosophy:

```bash
/grug-debug TypeError: Cannot read property 'id' of undefined
```

The implementation-guidance lenses (`grug`, `honest-code`, `tidy-first`, `parse-dont-validate`, `aposd`) activate automatically when their domain comes up. They cannot be invoked directly.

## Skills

### Review

Aggregate review across multiple lenses, parallelized.

#### `/review-all [scope] [lenses] [fix]`

Run code lens reviews in parallel and merge findings into a unified report. Default lenses: `grug`, `honest-code`, `tidy-first`, `parse-dont-validate`. Prefix a lens with `+` to add it to the defaults (`+aposd`, `+legacy-code`). Pass `fix` to apply non-conflicting findings after the review.

```bash
/review-all
/review-all src/auth.ts
/review-all grug honest-code
/review-all +aposd
/review-all fix
```

#### `/grug-review [scope]`

Review code for complexity demons through [grug brain developer](https://grugbrain.dev/) philosophy. Severity tiers: CLUB, CONCERN, GRUMBLE.

#### `/honest-code-review [scope]`

Review code for dishonest patterns using the [Honest Code](https://honestcode.software/) constructs (11 from honestcode.software, 1 extended from Gary Bernhardt's Functional Core, Imperative Shell). Severity tiers: CRIME SCENE, SUSPECT, WITNESS.

#### `/tidy-first-review [scope]`

Review code for tidying opportunities using [Tidy First?](https://www.oreilly.com/library/view/tidy-first/9781098151232/) philosophy. Severity tiers: TANGLED, CLUTTERED, DUSTY.

#### `/parse-dont-validate-review [scope]`

Review code for type-driven correctness using [Parse, Don't Validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/) and Make Illegal States Unrepresentable. Severity tiers: UNGUARDED, LEAKING, LOOSE.

#### `/aposd-review [scope]`

Review code for module depth, information hiding, and complexity using [A Philosophy of Software Design](https://web.stanford.edu/~ouster/cgi-bin/book.php) by John Ousterhout. Severity tiers: SHALLOW, EXPOSED, SURFACE.

#### `/legacy-code-review [scope]`

Review code for safe modification opportunities using [Working Effectively with Legacy Code](https://www.oreilly.com/library/view/working-effectively-with/0131177052/) by Michael Feathers. Severity tiers: UNTESTED, BRITTLE, RIGID.

### Debug

#### `/grug-debug <error or failing test>`

Debug problems through grug brain philosophy: small repro, real evidence, one change at a time. Accepts a bug description, error message, or failing test as the argument.

```bash
/grug-debug TypeError: Cannot read property 'id' of undefined
/grug-debug spec/models/user_spec.rb is failing
```

### Auto-triggered implementation guidance

These activate from conversation context when their domain comes up. They cannot be invoked directly.

| Skill | Activates on |
|-------|--------------|
| `grug` | Implementing features, refactoring, designing architecture, choosing abstractions, "keep it simple", "too complex", "complexity demon" |
| `honest-code` | "honest code", "declarative", "pure functions", "no classes", "no inheritance", "let it crash", "functional core", "imperative shell", "push effects" |
| `tidy-first` | "tidy", "clean up before", "refactor before", "messy code", "structural change", "reading order", "guard clauses", "extract helper", "dead code" |
| `parse-dont-validate` | "parse don't validate", "illegal states", "unrepresentable", "type-driven", "branded type", "smart constructor", "parse at boundary" |
| `aposd` | Designing modules, interfaces, error handling; questions about deep modules, information hiding, complexity symptoms |

## Contributing

To work on the plugin source, see [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT. See [LICENSE](LICENSE).
