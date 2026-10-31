---
name: data-oriented-programming
description: >-
  Apply Yehonathan Sharvit's Data-Oriented Programming principles while
  designing how a program represents and changes data: separate code from
  data, represent data with generic data structures, treat data as
  immutable, and separate the data schema from the data. Use when the user
  mentions "data-oriented programming", "DOP", "data as data", "generic data
  structures", "immutable data", "data schema", or "separate code from data",
  or when designing data models, entities, or the data a service passes
  around.
user-invocable: false
---

# Data-Oriented Programming

Apply the four principles of Data-Oriented Programming by Yehonathan Sharvit (*Data-Oriented Programming*, Manning, 2022) to the current task. Sharvit summarizes them in [Principles of Data-Oriented Programming](https://blog.klipse.tech/dop/2022/06/22/principles-of-dop.html) and explores each in a book excerpt linked below. He states that the principles are language-agnostic: they can be followed or broken in OOP languages such as Java and C#, in FP languages such as Clojure, and in languages that mix both.

The explanations below are this package's short summaries. Cite a principle by its number (`DOP #2`). For examples, benefits, and costs, follow the linked excerpt.

## The Principles

### 1. Separate code from data

[Excerpt](https://blog.klipse.tech/databook/2022/06/22/separate-code-from-data.html). Code lives in functions whose behavior does not depend on data held in the function's context. In OOP this means static methods operating on data classes that only hold data. In FP the principle is broken by hiding state in a function's lexical scope, such as a closure over mutable data.

Sharvit names the benefits as reuse in different contexts, testing in isolation, and less complex systems. He names the costs as no control over which code accesses which data, no packaging of code with its data, and more entities in the system.

### 2. Represent data with generic data structures

[Excerpt](https://blog.klipse.tech/databook/2022/06/22/generic-data-structures.html). Represent application data with maps (dictionaries) and arrays (lists), and with sets, trees, or queues when needed, instead of a specific class for each entity. Generic structures let the language's and libraries' rich set of collection functions work on any entity, and let the data model change shape without new classes.

Sharvit names the costs as a slight performance hit, no data schema, no compile-time check that data is valid, and, in some statically typed languages, explicit type casting.

### 3. Data is immutable

[Excerpt](https://blog.klipse.tech/databook/2022/06/22/immutable-data.html). Data never changes. A change creates a new version of the data, and a reference may be pointed at the new version. Efficient immutability usually needs persistent data structures, which some languages provide and others get from a library.

Sharvit names the benefits as passing data to any function with confidence, predictable behavior, fast equality checks by reference, and concurrency safety. He names the costs as a performance hit and the need for a persistent data structure library where the language lacks one.

### 4. Separate data schema from data representation

[Excerpt](https://blog.klipse.tech/databook/2022/06/22/data-validation.html). The expected shape of data is itself data, a schema (JSON Schema in his examples) kept apart from the data it describes. A validation library checks data against the schema where validation is wanted, and the developer chooses which data gets a schema.

Sharvit names the benefits as freedom to choose what to validate, natural optional fields, advanced validation conditions, and generated data model visualizations. He names the costs as a weak connection between data and its schema, and a light performance hit.

## Application Rules

- Cite principles by number. When suggesting a change, name the principle, what the current code does instead, and the alternative the principle describes.
- Weigh the costs Sharvit names. A principle whose cost outweighs its benefit in the code at hand is worth stating, with the reason, rather than applying anyway.
- Adapt the mechanism to the stack, following `lenses-fix/references/stack-adaptation.md`. In a statically typed language, generic data can mean a dictionary type, a JSON document type, or a record with no behavior, and schema validation can mean a JSON Schema validator. Do not claim compile-time guarantees that generic data cannot give.
- When a framework or project convention requires domain classes (for example ORM entities), report the code as a departure from DOP #2 with its reason, not as conformance.
- Direct users to Sharvit's book and excerpts for the full reasoning.

## Composition With Other Lenses

These notes are this package's guidance, not part of Sharvit's principles.

- **Honest Code:** P02 (pure functions over methods, data with no behavior) and P03 (I/O at the boundary) agree with DOP #1. P11 (declared types and schemas over imperative validation) agrees with DOP #4.
- **Parse Don't Validate:** both lenses check data at the boundary. Parse Don't Validate wants the parsed result to be a distinct type that carries the proof. DOP #2 keeps the result generic and DOP #4 keeps its shape in a schema. In a dynamically typed language the two agree: a parse function returns a generic map that a schema has checked. In a statically typed language they disagree about the representation, so `/lenses-fix` reports the tension instead of choosing.
- **APOSD:** APOSD hides data structures behind module interfaces, while DOP #1 lets any function read any data, a cost Sharvit names himself.
