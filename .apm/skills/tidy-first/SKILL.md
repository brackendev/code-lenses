---
name: tidy-first
description: >-
  Apply Tidy First? philosophy when changing existing code. Use when user says
  "tidy", "tidy first", "clean up before", "refactor before", "messy code",
  "before I change this", "structural change", "reading order", "guard clauses",
  "extract helper", "dead code", or when preparing code structure before a
  behavioral change.
user-invocable: false
---

# Tidy First? Philosophy

Apply the [Tidy First?](https://www.oreilly.com/library/view/tidy-first/9781098151232/) philosophy by Kent Beck (O'Reilly, 2023) to the current task. Core rule: separate structural changes from behavioral changes.

## The Cardinal Rule

Structural changes (tidyings) and behavioral changes go in separate commits. A tidying changes structure without changing behavior. A behavioral change changes what code does without restructuring. If a commit does both, split it.

## The 15 Tidyings

Apply these structural moves to prepare code for behavioral change:

1. **Guard clauses** -- Replace nested if-else with early returns. Flatten control flow so the main path reads straight down.
2. **Explaining variables** -- Extract complex expressions into named variables that reveal intent.
3. **Explaining constants** -- Replace magic numbers and strings with named constants.
4. **Explicit parameters** -- Replace implicit state (globals, configuration lookups) with function parameters.
5. **Chunk statements** -- Group related statements with blank lines to create visual paragraphs.
6. **Extract helper** -- Pull a cohesive block of code into a named function.
7. **One pile** -- Inline overly fragmented code back into one place, then re-extract with better boundaries.
8. **Dead code** -- Delete code that is never executed. Verify with search before deleting.
9. **Normalize symmetries** -- Make structurally similar code use identical patterns so differences stand out.
10. **New interface, old implementation** -- Create the interface you want and delegate to the existing code.
11. **Reading order** -- Reorder declarations so readers encounter them top-down, callers before callees.
12. **Cohesion order** -- Move related code closer together: functions that call each other, data that changes together.
13. **Move declaration and initialization together** -- Declare variables where they are first used, not at top of scope.
14. **Remove unnecessary comments** -- Delete comments that restate what the code already says.
15. **Eliminate needless complexity** -- Remove abstractions, parameters, or indirection that serve no current purpose.

## When to Tidy

Choose timing based on the relationship between the tidying and the next behavioral change:

- **Tidy first:** The tidying makes the next behavioral change easier to write, review, or understand. This is the most common case.
- **Tidy after:** The behavioral change is done but revealed structural debt. Tidy in a follow-up commit.
- **Tidy later:** The tidying is real but not blocking current work. Note it as a TODO or issue.
- **Tidy never:** The code works, nobody needs to change it, and tidying risks introducing bugs. Leave it alone.

Decision heuristic: "Will this tidying make the next change easier? If yes, tidy first. If the code area is cold (rarely changed), tidy never."

## Structure Enables Behavior

Good structure makes behavioral changes small, safe, and obvious. Bad structure makes behavioral changes risky, scattered, and hard to review. Tidying is not gold-plating or cleaning up for fun. It is investment that pays off in the immediate next change. If the tidying does not make the next change easier, skip it. When structure is right, it can reveal behavioral changes that were not obvious before.

## What Tidy First Never Does

- Never mixes structural and behavioral changes in one commit.
- Never tidies code that is not about to change (speculative tidying).
- Never rewrites working code for aesthetic preference.
- Never introduces speculative abstraction layers. Tidyings add local, behavior-preserving structure (extracting a helper, wrapping an interface) that makes the next change easier.
- Never tidies without understanding the code's purpose first.

## Relationship to Other Skills

Grug provides anti-complexity instinct. Tidy First provides specific structural moves. When grug says "simplify," tidy-first provides the how: which tidying to apply. Both can apply to the same code.
