---
name: parse-dont-validate
description: >-
  Apply Parse Don't Validate philosophy when handling input, designing types, or
  working at system boundaries. Use when user says "parse don't validate",
  "illegal states", "unrepresentable", "type-driven", "branded type", "newtype",
  "smart constructor", "validate at boundary", "parse at boundary", or when
  designing data types, input handling, or domain models.
user-invocable: false
---

# Parse, Don't Validate

Apply [Parse, Don't Validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/) by Alexis King and the [Make Illegal States Unrepresentable](https://blog.janestreet.com/effective-ml-revisited/) principle (coined by Yaron Minsky; see also [Scott Wlaschin's explainer](https://fsharpforfunandprofit.com/posts/designing-with-types-making-illegal-states-unrepresentable/)) to every code change. Core rule: transform unstructured input into typed, validated representations at the boundary, then use those representations everywhere else.

## The Core Distinction

- **Validation** checks whether data is valid and returns a boolean or throws. The caller still holds the original unvalidated type and must remember the check happened.
- **Parsing** transforms data from a less structured type into a more structured type. If it succeeds, the result type guarantees the invariants hold. If it fails, the error is reported at the boundary.

Prefer parsing. After a successful parse, downstream code cannot encounter invalid data because the type system prevents it.

## Principles

### 1. Parse at the Boundary

**Instead of:** Validating fields repeatedly in business logic. Checking `if email != ""` in three different functions.
**Use:** A parse function at the system boundary (HTTP handler, CLI entry, file reader, message consumer) that converts raw input into a domain type. All code past the boundary receives the parsed type.

The boundary is where unstructured data enters the system: user input, API responses, database rows, file contents, environment variables, message queues.

### 2. Make Illegal States Unrepresentable

**Instead of:** A `User` type where `email` is `string` and might be empty, malformed, or missing.
**Use:** A `User` type where `email` is `Email` (a type that can only be constructed through parsing). If you have an `Email`, it is valid by construction.

Use the type system to encode invariants:

- **Sum types / discriminated unions** for states that are mutually exclusive (e.g., `Loading | Loaded | Error` instead of `{ loading: boolean, error: string | null, data: T | null }`).
- **Branded types / newtypes / smart constructors** for constrained primitives (e.g., `NonEmptyString`, `PositiveInt`, `Email`, `UserId`).
- **Required fields over optional** when the data is always present after parsing.

### 3. Let the Types Carry the Proof

**Instead of:** Returning `string` from a parse function and hoping callers remember it was validated.
**Use:** Returning a distinct type (`Email`, `ValidatedOrder`, `ParsedConfig`) so the type system enforces that only parsed data flows downstream.

A function signature like `processOrder(order: ValidatedOrder)` is self-documenting: the caller must parse first. A function signature like `processOrder(order: RawOrder)` invites bugs.

### 4. Fail Fast, Fail at the Boundary

**Instead of:** Letting invalid data travel deep into the system before a check catches it.
**Use:** Parsing at entry. If parsing fails, return an error immediately. No invalid data enters the core.

Combine with structured error reporting: collect all parse failures (not only the first) so the caller can fix everything at once.

### 5. Avoid Shotgun Validation

**Instead of:** Sprinkling `if (x < 0) throw` checks across the codebase wherever `x` is used.
**Use:** Parsing `x` into `PositiveInt` once at the boundary. Every function that accepts `PositiveInt` is guaranteed `x >= 1` without checking.

Shotgun validation is a symptom of unstructured types. The fix is a better type, not more checks.

## Language-Specific Techniques

| Language | Technique |
|----------|-----------|
| TypeScript | Branded types (`type Email = string & { __brand: 'Email' }`), Zod/ArkType schemas, discriminated unions |
| Python | `NewType`, `@dataclass` with `__post_init__` validation, Pydantic models |
| Rust | Newtype pattern (`struct Email(String)`), `TryFrom` implementations, `enum` for sum types |
| Go | Unexported struct fields with constructor functions, custom types wrapping primitives |
| Java/Kotlin | Value classes, sealed interfaces, records with validation in constructor |
| Swift | Enums with associated values, `RawRepresentable` with failable initializers |
| Clojure | `clojure.spec`/Malli schemas with `conform`/coercion, smart constructor functions returning domain maps or throwing `ex-info`, namespaced keys for domain concepts, tagged maps with `:type`/`:kind` keys for sum types |

## Application Rules

- When detecting raw primitives (`string`, `int`, `any`) used to represent domain concepts, suggest a domain type with a parse constructor.
- When detecting repeated validation of the same invariant, suggest parsing once at the boundary.
- When detecting states modeled with multiple booleans or nullable fields, suggest a sum type / discriminated union.
- Consider the project's language when suggesting types. The principle applies everywhere, but the mechanism differs.
- **Composition with APOSD:** APOSD says "define errors out of existence." Parse Don't Validate says "make invalid states unconstructable." These reinforce each other: parse at the boundary to eliminate errors, design deep module interfaces that accept only parsed types.
- **Composition with honest-code:** Honest Code Construct 10 says "Declare What, Not How." Parsed types are declarations of what valid data looks like. They complement each other.
