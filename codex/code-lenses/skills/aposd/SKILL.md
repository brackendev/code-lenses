---
name: aposd
description: >-
  Apply A Philosophy of Software Design principles when designing modules,
  interfaces, or error handling.
user-invocable: true
---

# A Philosophy of Software Design

When invoked, apply principles from [A Philosophy of Software Design](https://web.stanford.edu/~ouster/cgi-bin/book.php) by John Ousterhout (2nd edition, 2021) to the current implementation task. The core mission: manage complexity through deep modules, information hiding, and strategic design.

## Complexity Defined

Three symptoms of complexity:

1. **Change amplification:** A simple change requires modifying many places.
2. **Cognitive load:** Developer must hold too much context to make a change safely.
3. **Unknown unknowns:** It is not obvious what needs to change or what might break.

Two causes of complexity:

1. **Dependencies:** Code in one place forces awareness of code in another.
2. **Obscurity:** Important information is not obvious from the code structure.

When these symptoms appear, the design has a complexity problem.

## Deep Modules

A module's cost is its interface. Its value is its functionality.

- **Deep module:** Simple interface, powerful implementation. High value-to-cost ratio.
- **Shallow module:** Complex interface relative to the functionality it provides. Low value-to-cost ratio.
- Prefer fewer, deeper modules over many shallow ones.
- Each module should provide a meaningful abstraction that hides implementation complexity.
- Warning signs of shallow modules: pass-through methods, classes that add no new abstraction, interfaces that mirror the implementation.
- Aim for interfaces that are obvious to use without reading the implementation.

## Information Hiding

Each module should encapsulate design decisions behind its interface.

- Hidden information includes data structures, algorithms, low-level details, and error recovery.
- **Information leakage:** When the same design decision appears in multiple modules, changes to that decision require changing all of them. This is the most common cause of change amplification.
- **Temporal decomposition** (splitting code by execution order rather than by information) often causes leakage. Group by what information is needed, not when it is used.
- If two modules share knowledge of the same design decision, consider merging them or extracting the shared knowledge into a new module.

## Pull Complexity Downward

When complexity is unavoidable, push it into the implementation, not the interface.

- A module's implementer deals with complexity once. Every caller deals with interface complexity on every call.
- Prefer a module that is harder to implement but easier to use.
- Configuration parameters are a sign of pushing complexity upward. Provide good defaults instead.

## Define Errors Out of Existence

Exceptions are a major source of complexity. Each exception requires handling in every caller up the stack.

- **Define errors out of existence:** Change the interface so the error condition cannot occur.
- Example: Instead of throwing on "file not found" in delete, make delete succeed if the file does not exist (idempotent).
- When errors cannot be eliminated: handle them at the lowest level possible. Do not propagate exceptions unless the caller can do something meaningful.
- **Exception aggregation:** Handle many low-level errors in one place rather than distributing handling across the codebase.

## General-Purpose Interfaces

Design interfaces that are somewhat general-purpose, even for specific use cases.

- A general-purpose interface is simpler (fewer special cases) and more reusable.
- The interface should reflect the problem domain, not the current use case's specific requirements.
- "Somewhat general-purpose" means the current use case plus a few foreseeable variations. Not a framework.
- The implementation can be specific. Only the interface needs to be general.

## Strategic vs Tactical Programming

- **Tactical programming:** Get the feature working as fast as possible. Accumulates complexity debt.
- **Strategic programming:** Invest a small amount of extra time in each change to improve the design. The investment compounds.
- The goal is not perfection. The goal is a little better with every change.
- A 10-20% investment in design per task is sufficient.
- Tactical programming feels faster but slows the team within months.

## Application Rules

- When designing a new module, evaluate interface depth. Simple interface with powerful implementation is the goal.
- When information leaks between modules, restructure to hide the shared decision.
- When error handling spreads across many layers, define errors out of existence or aggregate.
- APOSD provides module-level design guidance. The grug skill provides anti-complexity instinct. Both can apply to the same code.
- **Error handling composition with honest-code:** Honest Code says "raise at source, handle at boundary." APOSD says "define errors out of existence." Apply APOSD first: if the interface can be redesigned so the error cannot occur, do that. When errors remain, apply honest-code: raise at source, handle at the boundary, never swallow.
- For the full treatment, see "A Philosophy of Software Design" by John Ousterhout.
