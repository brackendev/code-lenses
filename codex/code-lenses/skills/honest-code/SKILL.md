---
name: honest-code
description: >-
  Trigger on: "honest code", "declarative", "pure functions", "flat data", "no
  classes", "no inheritance", "no state", "let it crash", "dishonest",
  "functional core", "imperative shell", "side effects", "push effects", or
  when user asks about Honest Code constructs. Apply the Honest Code
  constructs during implementation (11 from honestcode.software, 1 extended
  from Gary Bernhardt).
user-invocable: true
---

# Honest Code

Apply the [Honest Code](https://honestcode.software) constructs to every code change. Constructs 1 through 11 are by Adam Zachary Wasserman. Construct 12 extends the philosophy with Gary Bernhardt's Functional Core, Imperative Shell pattern. Honest software uses constructs that tell the truth about what they do.

## The Constructs

Follow these constructs when writing or changing code. When you detect an "instead of" pattern, suggest the corresponding "use" alternative. Reference the construct by name.

### 1. Data Is Data (Ch 3)

**Instead of:** Class with fields and methods bundled together.
**Use:** TypedDict, record, struct, or plain object with no behavior attached.

Keep data structures flat. If you cannot serialize it to JSON, it is too complex. Separate data from the functions that operate on it.

### 2. Input In, Output Out (Ch 4)

**Instead of:** Methods that mutate `self` or `this`. Class hierarchies for polymorphism.
**Use:** Pure functions. Dict-lookup polymorphism (`HANDLERS[type](data)`). Pass state as parameter, return new state.

Test becomes: `assert f(input) == expected`. No setup, no teardown, no mocking.

### 3. One Source of Truth (Ch 5)

**Instead of:** Client-side state synchronized with server state (Redux, MobX, Zustand).
**Use:** DOM or database as the single source. Server-rendered HTML with targeted swaps (HTMX, Turbo).

When two sources disagree, users experience bugs. Eliminate one source.

### 4. Declare, Don't Instruct (Ch 12)

**Instead of:** Imperative DOM manipulation. Event listeners wiring behavior to elements. JavaScript formatting libraries.
**Use:** HTML attributes (HTMX attributes, data attributes). Browser-native APIs (`Intl` for formatting). Declarative bindings.

Let the platform do the work. Declarations are easier to read, test, and maintain than instructions.

### 5. Compose Flat, Never Deep (Ch 6)

**Instead of:** `class B extends A`. Handler hierarchies. Middleware stacks.
**Use:** `pipe(a, b, c)`. Independent functions composed at call site. Hooks and dependency injection.

Each function does one thing. Composition happens at the edges, not through inheritance chains.

### 6. Let It Crash (Ch 7)

**Instead of:** `catch (e) { return false }`. Nested try-catch. Inline retry logic.
**Use:** Raise at source, handle at boundary. Typed exceptions. Supervision (let supervisor decide). Task queue with retry policy.

Swallowed errors are lies. Crash early, crash loud, let the caller or supervisor decide what happens next.

### 7. Profile First, Fix Architecture (Ch 8)

**Instead of:** Application-level object cache. Three service calls where one query works.
**Use:** Single SQL join under 3ms. Proper indexes. Measure before caching.

Caching without profiling is premature architecture. Most performance problems are query problems.

### 8. Boring Tests (Ch 9)

**Instead of:** 9 `@MockBean` annotations. Mock store + 4 wrapper components to test a UI.
**Use:** `assert f(input) == expected`. HTTP request + check HTML. Gherkin for features.

If the test is interesting, the design is the bug. Honest architecture makes tests boring.

### 9. Constrain AI (Ch 10)

**Instead of:** Accepting AI-generated 500-line classes. Reviewing sprawling output.
**Use:** Honest architecture as the prompt. Five 10-line functions instead of one 500-line class.

AI generates the most statistically common code. That code is a crime scene. Constrain with honest architecture: review cost drops from an hour to five minutes.

### 10. Declare What, Not How (Ch 11)

**Instead of:** Mutable variables and side effects. Imperative validation.
**Use:** Declarations (what) over instructions (how). Pure functions and typed data. Type declarations enforced by the system.

Let the type system and runtime enforce constraints instead of writing validation code.

### 11. Rescue, Don't Rewrite (Ch 13)

**Instead of:** Big-bang rewrite. "We need to start over."
**Use:** Extract one pure function this sprint. Strangler pattern: wrap old, build new alongside. Show faster tests, fewer bugs. Find one ally. Let it spread.

Rewrites fail. Incremental rescue succeeds.

### 12. Push Effects to the Edges

**Instead of:** Business logic interleaved with database calls, HTTP requests, file I/O, and logging. Functions that are half calculation, half side effect.
**Use:** Functional core: pure functions that take data and return data. Imperative shell: a thin outer layer that performs I/O, calls the core, and writes results.

Structure each feature as: read inputs (shell) -> compute decision (core) -> write outputs (shell). The core is easy to test (`assert f(input) == expected`) because it has no I/O. The shell is thin enough to verify by inspection. Based on Gary Bernhardt's [Boundaries](https://www.destroyallsoftware.com/talks/boundaries) talk. Extends Constructs 2 and 3 from function-level purity to architecture-level separation.

## Application Rules

- When detecting code that matches an "instead of" pattern, suggest the corresponding "use" pattern. Reference the construct by name and number.
- Consider the project's language when suggesting alternatives. The constructs apply across Python, TypeScript, Go, Java, and others.
- Honest Code provides specific construct-level guidance. The grug skill provides general anti-complexity instinct. Both can apply to the same code.
- Direct users to [honestcode.software](https://honestcode.software) for full examples and reasoning.
