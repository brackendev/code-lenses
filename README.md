# Code Lenses

Code Lenses is a set of review, debugging, and implementation-guidance skills for AI coding agents. It packages the same core lenses for Claude Code and Codex using the [Agent Skills](https://agentskills.io) format.

## What It Is

Code Lenses helps an agent review or write code through six design lenses:

- `grug`: keep implementations simple, local, and pragmatic ([The Grug Brained Developer](https://grugbrain.dev/))
- `aposd`: prefer deep modules and strong information hiding ([A Philosophy of Software Design](https://web.stanford.edu/~ouster/cgi-bin/book.php) by John Ousterhout)
- `honest-code`: favor flat data, pure functions, and honest constructs ([Honest Code](https://honestcode.software) by Adam Zachary Wasserman, with [Functional Core, Imperative Shell](https://www.destroyallsoftware.com/talks/boundaries) by Gary Bernhardt)
- `tidy-first`: separate structural cleanup from behavioral change ([Tidy First?](https://www.oreilly.com/library/view/tidy-first/9781098151232/) by Kent Beck)
- `parse-dont-validate`: parse at boundaries, make illegal states unrepresentable ([Parse, Don't Validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/) by Alexis King)
- `legacy-code`: find seams, write characterization tests, modify untested code safely ([Working Effectively with Legacy Code](https://www.oreilly.com/library/view/working-effectively-with/0131177052/) by Michael Feathers) (review-only, no auto-applied implementation skill)

The packaged skills are split into two groups:

- Review and debug skills you invoke directly
- Auto-applied implementation skills that shape how the agent approaches code changes

## Platforms

| Platform | Package Path | Marketplace Metadata | Guide |
|---------|--------------|----------------------|-------|
| Claude Code | `./claude/code-lenses` | `./.claude-plugin/marketplace.json` | [Claude README](./claude/README.md) |
| Codex | `./codex/code-lenses` | `./.agents/plugins/marketplace.json` | [Codex README](./codex/README.md) |

## Bundled Lenses

### Review and debug skills

| Skill | Purpose |
|-------|---------|
| `review-all` | Run review lenses and aggregate findings (5 default, Legacy Code opt-in) |
| `grug-review` | Review for complexity, over-engineering, and abstraction debt |
| `aposd-review` | Review for module depth, information hiding, and complexity symptoms |
| `honest-code-review` | Review for dishonest patterns using the Honest Code constructs |
| `tidy-first-review` | Review for tidying opportunities and mixed structural and behavioral changes |
| `parse-dont-validate-review` | Review for type-driven correctness, boundary parsing, and illegal states |
| `legacy-code-review` | Review for missing tests, seams, and safe modification of untested code |
| `grug-debug` | Debug through small repro, evidence, and one-change-at-a-time workflow |

### Auto-applied implementation skills

| Skill | Purpose |
|-------|---------|
| `grug` | Keep implementations simple, local, and pragmatic |
| `aposd` | Apply deep modules and information-hiding principles |
| `honest-code` | Prefer flat data, pure functions, and honest constructs |
| `tidy-first` | Separate structural and behavioral change and tidy only where it helps |
| `parse-dont-validate` | Parse at boundaries, make illegal states unrepresentable, use domain types |

## How To Use It

Choose the platform guide that matches your agent:

- [Claude README](./claude/README.md)
- [Codex README](./codex/README.md)

Both packages expose the same core lenses. The main difference is how they are invoked:

- Claude Code uses installed plugin commands such as `/code-lenses:review-all`
- Codex uses the packaged skills directly in prompts such as `Use review-all on the current diff.`

## Repository Layout

| Path | Contents |
|------|----------|
| `claude/code-lenses/` | Claude package: manifest, reviewer agents, and skills |
| `codex/code-lenses/` | Codex package: manifest and skills |
| `.claude-plugin/marketplace.json` | Repo-level Claude marketplace entry |
| `.agents/plugins/marketplace.json` | Repo-level Codex marketplace entry |

## Notes

- This repo packages and distributes the Code Lenses plugin for Claude Code and Codex.
- The skill content follows the Agent Skills format, so other compatible tools can reuse the `SKILL.md` directories if they integrate them separately.
- Debug skills exist only for grug (`grug-debug`). The other lenses are review and implementation focused. Parse Don't Validate and Legacy Code do not have dedicated debug skills.
