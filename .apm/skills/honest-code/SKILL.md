---
name: honest-code
description: >-
  Apply the Honest Code Principles (P01-P25) while writing or changing code:
  lookup polymorphism, pure functions over methods, I/O at the boundary,
  composition over inheritance, typed exceptions at the boundary, no implicit
  defaults, dispatch tables, and faults returned to the caller. Use when the
  user asks for honest or declarative code, or mentions pure functions, side
  effects, avoiding classes, inheritance, or state, mocks, implicit defaults,
  "dishonest" code, or "functional core, imperative shell".
user-invocable: false
---

# Honest Code

Apply the Honest Code Principles to the current task. The principles are maintained by the Open Honest Foundation (Adam Zachary Wasserman) as a single source file, and this skill ships an unchanged copy of it at a pinned commit: `references/honest-code-principles.md`. Its header records the upstream URL, commit, and license.

Read `references/honest-code-principles.md` before applying the lens. The file is the rule text. This skill cites principles by number and does not restate them. The upstream README asks projects not to copy the file and not to summarize it, because copies drift. This package ships the exact file anyway so that the lens works without network access, and it keeps the copy pinned to one commit.

## Application Rules

- Cite a principle by its number (`P15`), never by its title alone. Upstream keeps each number permanently and has renamed titles.
- Each principle names the category of defect it removes. When suggesting a change, name that defect for the code at hand, then the honest alternative the principle gives.
- Apply the principle as written. When a project or framework constraint prevents it (for example, an ORM that requires entity classes), say that the code departs from the principle and why. Do not describe the departure as conforming, and do not restate the principle in weaker terms.
- P05 (DOM as State), P06 (HTML attributes), and P22 (declarative lifecycle) concern browser interfaces. When the work has no browser interface, they do not apply. When it does and uses another client-state design, they apply, and the disagreement is stated.
- P02 says how to write honest code in Java and C#, which do not allow standalone functions. Follow that text rather than inventing another translation.
- The "Enforced by" lines name upstream's own checkers (`honest-check`, `honest-test`, `feature-gate.sh`). They describe upstream's tooling. Do not claim a project is checked by them unless it runs them.

## Related Background

Gary Bernhardt's [Boundaries](https://www.destroyallsoftware.com/talks/boundaries) talk (Functional Core, Imperative Shell) argues for the same separation as P03. It is background reading, not one of the principles.

The book *Honest Code* (<https://honestcode.software>) gives the longer reasoning and examples. When the book and the principles file disagree, follow the principles file, which upstream calls the single source.

## Composition With Other Lenses

These notes are this package's guidance, not part of the principles.

- The grug skill provides general instinct against complexity. Both can apply to the same code.
- Parse Don't Validate and P11 and P14 agree on validating once at the boundary. Parse Don't Validate adds parsed domain types, which P02's plain data can carry when only a parse function produces them.
- APOSD prefers defining errors out of existence. Where an error remains, P08 and P24 govern how it travels.
