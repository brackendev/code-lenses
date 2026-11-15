---
name: simple-made-easy
description: >-
  Apply the design ideas of Rich Hickey's talk Simple Made Easy while
  choosing constructs and designing components: tell simple (not entangled)
  apart from easy (familiar), prefer values to state, plain data to objects,
  and open polymorphism to inheritance and repeated branching, and keep
  what, who, when, and why separate. Use when the user mentions "Simple Made
  Easy", "Rich Hickey", "complect", "complecting", "decomplect", "simple vs
  easy", "simple versus easy", "entangled", or "untangle", or asks which
  concerns a construct or a design ties together.
user-invocable: false
---

# Simple Made Easy

Apply the design ideas of Rich Hickey's talk [Simple Made Easy](https://www.infoq.com/presentations/Simple-Made-Easy/) (Strange Loop, 2011) to the current task. The talk separates simple, which means not entangled with something else, from easy, which means familiar or close at hand. It calls the entangling "complecting", lists constructs that entangle and simpler replacements for them, and describes how to design an abstraction by separating what, who, how, when, where, and why.

The talk has no numbered list. The numbering, the grouping, and the explanations below are this package's short summaries, and they quote nothing from the talk. Cite a principle by its number (`SME #3`). For the reasoning, watch the talk.

## The Principles

### 1. Simple is not easy

Simple is a property of the construct: it has one role and is not interleaved with another. Easy is relative to a person: familiar, already installed, or quick to write. Judge a construct by what it leaves in the program that must be run, debugged, and changed, not by how it feels to write. A construct that is easy to start with can leave an entangled result.

### 2. Compose, do not entangle

Complexity comes from interleaving things that could stand apart. Build from parts that know nothing about each other beyond a narrow abstraction, and place them side by side. Dividing code into modules or classes does not achieve this while one part still relies on an unstated detail of another. Simplicity is not a count: separating concerns often produces more parts, and more independent parts are easier to change than fewer entangled ones.

### 3. Values over state

State ties a value to time. The same question can return a different answer later, and that uncertainty spreads to everything that touches the state. Use immutable values by default. Where state is needed, keep it behind a reference that can give a consistent value, take the value, and pass the value on instead of the reference. Where several values must agree, change them together in a transaction.

### 4. Plain data for information

Information is simple by nature. Represent it with the general data structures of the language (maps, sets, and sequences) instead of a class for each kind of information. Generic code can then process any of it, and logic is not tied to one representation.

### 5. Functions and namespaces over methods

A method ties a function to state, and often to a class that also acts as its namespace. Prefer stateless functions that receive their data as arguments, grouped by a namespace or module mechanism that does naming and nothing else.

### 6. Open polymorphism over inheritance and repeated branching

Inheritance ties types to each other. A switch or a pattern match ties every place that uses it to the full, closed list of variants. Prefer a mechanism in which the data types, the sets of operations, and the connection between them are defined independently, so that one can be added without editing the others.

### 7. Data over syntax

Syntax ties meaning to order and position. Describe things with data that programs can build, inspect, and transform, both inside a program and in what programs send to each other, instead of with special syntax or free-form text that needs a parser.

### 8. Declare what, leave how

A loop ties what is computed to how the iteration proceeds. Object-relational mapping ties logic to a representation. Conditionals spread through a program tie its rules to its structure. Prefer collection and set operations, declarative queries, and rules gathered in one declarative place.

### 9. Separate what, who, how, when and where, and why

Design an abstraction by answering each question apart from the others. Abstraction here means drawing away what does not matter, which is not the same as hiding it.

- **What:** small sets of function specifications that take and return values or other abstractions, much smaller than is common.
- **Who:** the components. Build a component from subcomponents that it receives as arguments instead of fixing them inside, and expect more and smaller components.
- **How:** the implementations, connected through the polymorphism mechanism and kept apart from each other. An abstraction should not dictate how it is implemented.
- **When and where:** a component that calls another directly is tied to where the other is and to when the call happens. A queue between them removes both ties.
- **Why:** the policy and rules, kept out of the flow of the application and stated in a declarative form.

## Application Rules

- Cite principles by number. When suggesting a change, name the concerns the current construct ties together, say how one could change without the other, and give the alternative.
- The talk presents its simpler choices as simpler, not as free of cost, and allows for reasons to depart from them. State the cost that applies to the code at hand: more parts, more indirection, a performance cost, or a construct the team does not know. A principle whose cost outweighs its benefit there is worth stating, with the reason, rather than applying anyway.
- Do not add a part for a variation that does not exist. SME #2 separates concerns that are entangled in the code today. It does not justify an interface, a queue, or an extension point for a need that nobody has. This rule is this package's guidance.
- Adapt the mechanism to the stack, following `lenses-fix/references/stack-adaptation.md`. Open polymorphism can mean small interfaces, traits, type classes, protocols, or a table of functions. Values can mean immutable records, read-only fields, or a persistent collection library. A queue can mean a channel, a message queue, an event stream, or work handed to an executor, a scheduler, or a task runner that the component already has. Do not recommend imitating a mechanism the language lacks.
- When a framework or an established project convention requires the entangling construct (for example ORM entities, class-based components, or a required base class), report the code as a departure from the cited principle with its reason, not as conformance.
- Direct users to the talk for the full reasoning.

## Composition With Other Lenses

These notes are this package's guidance, not part of the talk.

- **Data-Oriented Programming:** DOP #1 agrees with SME #5, DOP #2 with SME #4, and DOP #3 with SME #3. When a finding concerns data representation alone, cite the DOP principle as well. SME adds dispatch, syntax, and the connections between components, which DOP does not cover.
- **Honest Code:** P02 (pure functions over methods, data with no behavior) agrees with SME #4 and #5. P04 (composition over inheritance) agrees with SME #6. P01 (lookup polymorphism) replaces conditional chains with a table, which agrees with SME #6 and #8. P16 (dispatch tables close open input) wants the set of keys written out in one place, while SME #6 wants variants added without editing a central place, so `/lenses-fix` reports the tension instead of choosing.
- **Grug:** both lenses oppose complexity, but they measure it differently. Grug prefers fewer parts, local behavior, and no abstraction before the third repetition. SME #2 and #9 accept more parts, smaller interfaces, and a queue between components when they separate concerns that are entangled today. SME #1 also warns that the familiar choice is not always the simple one.
- **APOSD:** APOSD prefers fewer, deeper modules with somewhat general-purpose interfaces that hide data structures. SME #9 prefers many small sets of functions and more components, and SME #4 leaves information as plain data that any function can read.
- **Parse Don't Validate:** Parse Don't Validate wants the parsed result to be a distinct type that carries the proof, and SME #4 keeps information in general data structures. In a dynamically typed language the two agree: a parse function returns checked plain data. In a statically typed language they disagree about the representation.
