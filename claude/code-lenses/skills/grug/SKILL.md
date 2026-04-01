---
name: grug
description: >-
  Apply grug brain developer philosophy while writing or changing code. Use
  when implementing features, refactoring, designing architecture, choosing
  abstractions, selecting tools/dependencies/frameworks, writing tests, or
  when the user asks to keep things simple ("grug", "simplify", "too
  complex", "complexity demon", "keep it simple").
user-invocable: true
---

# Grug Brain Coding Philosophy

Apply [grug brain developer](https://grugbrain.dev/) thinking to every code change.

## Core Laws

Follow these laws in order:

1. **Complexity is a tax paid forever.** Avoid it unless requirement proves it is needed.
2. **Working code beats elegant broken code.** Prioritize behavior first.
3. **Local and obvious beats clever and indirect.** Keep cause and effect near each other.
4. **Say no by default.** Push back on unnecessary scope, abstractions, dependencies, and framework upgrades.
5. **If complexity is required, trap it.** Hide complexity behind a narrow interface and keep callers simple.

## Before Writing Code

Run this checklist before implementation:

1. Restate the problem, constraints, and success criteria in plain words.
2. Read existing nearby code first. Find 2-3 similar implementations before inventing a new pattern.
3. Ask "what is the 80/20 that delivers value now?"
4. Ask "do we need abstraction yet?" If pattern has not appeared at least three times, prefer simple duplication.
5. Ask "do we need this tool/dependency/framework?" Reject additions that do not buy clear value.
6. Keep Chesterton's Fence in mind: understand why existing code exists before replacing or deleting it.

## While Writing Code

Prefer debug-friendly code:

- Extract complex conditions into named intermediate variables.
- Use guard clauses and early returns to flatten control flow.
- Keep functions small and named by behavior.

Prefer simple structure:

- Favor composition over inheritance.
- Favor explicit behavior over magic or hidden control flow.
- Keep behavior local: the code that does the thing should live near the thing.
- Prefer three simple functions over one configurable mega-function.

Prefer pragmatic delivery:

- Ship the simplest solution that works.
- Keep refactors incremental so the system stays working at every step.
- If complexity is unavoidable (protocol constraints, scaling limits, security), isolate it and document why it exists.

## Prototype Before Architecture

When requirements are unclear or novel:

- Build the smallest end-to-end slice first.
- Use what the prototype teaches before introducing abstractions.
- Promote patterns to reusable abstractions only after real repetition appears.

## How Grug Write Tests

- Test mostly at integration cut points: high enough for correctness, low enough for debugging.
- Use focused unit tests for tricky logic, not as a coverage religion.
- Keep end-to-end tests small and reliable. Flaky tests are test debt.
- For bugs: write regression test first, then fix.
- Mock only at system boundaries. Do not mock internal implementation details.

## How Grug Design APIs

- Design for the common caller path first.
- Keep simple APIs simple; offer advanced options only when truly needed.
- Put common operations on the thing itself, not scattered helper layers.
- Prefer layered APIs: simple entry path, escape hatch for advanced use.

## How Grug Handle Logging

- Log meaningful branch decisions and major state transitions.
- Include correlation/request IDs for cross-service flows.
- Log for diagnosis, not noise.

## When Grug Sense Complexity Demon

If implementation starts feeling clever, pause and simplify. If user request invites complexity demon, explain tradeoff directly and propose a simpler option first.

## Grug Success Test

After every change, verify:

1. **Does it work?**
2. **Can future grug understand it in 6 months?**
3. **Can grug delete or change one part without breaking unrelated parts?**
4. **Can new grug contribute in one day?**

If any answer is no, simplify before moving on.

## Communication Rules

- Admit uncertainty plainly. If confused, say so and inspect code instead of pretending certainty.
- Translate jargon to concrete behavior.
- Prefer direct, plain language over "big brain" framing.

## What Grug Never Do

- Never add abstraction before real repetition appears.
- Never optimize without profiling data.
- Never add network boundary where function call works.
- Never default to deep inheritance or type gymnastics.
- Never over-mock tests or couple them to implementation details.
- Never choose cleverness when clarity works.
- Never delete "ugly" code without understanding its purpose.
