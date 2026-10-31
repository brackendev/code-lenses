# Stack Adaptation

Part of `/lenses-fix`. Every review skill in this package applies its lens to the reviewed code's stack by these rules, whether `/lenses-fix` runs it or the user invokes it directly. The rules are this package's guidance, not part of any lens's source. They never name a stack: the reviewer identifies the stack from the code and applies the rules to it.

## 1. Identify each component

A component is a group of files in scope that share a language, framework, and role, such as a C# web API project, a ClojureScript front end, or a Python worker. For each component, record what the code shows, with file references:

| Property | Evidence |
|---|---|
| Language and framework | Build and dependency files (`*.csproj`, `deps.edn`, `package.json`, `pyproject.toml`, and others), imports, and source extensions |
| Paradigm | Whether classes or objects own mutable state and the methods that change it, or functions transform plain data passed to them |
| Typing | Whether a compiler or type checker enforces types (static), or types are checked only at run time (dynamic) |
| Data representation | Specific classes or records per entity, or generic maps and lists |
| UI architecture | Server-rendered HTML, a client-side single-page app, or no browser interface |
| Tests | Whether tests cover the code the change touches |

Judge each property from the code, not from the language alone. A C# component can be written in a functional style, and a Clojure component can be built on records and protocols. When the evidence is thin or the component mixes styles, write "mixed" or "unknown" instead of guessing.

## 2. Translate the mechanism, keep the prescription

Each lens prescribes an outcome and shows a mechanism in its source's language. Keep the outcome and use the component's idiomatic mechanism for it. Examples:

- Honest Code P02 tests plain data with `json.dumps()`. Use the component's own serialization format as the test: EDN in Clojure, `System.Text.Json` in C#.
- Honest Code P01 and P16 show a Python dict of handlers. A map of functions, a multimethod, or a dictionary of delegates is the same dispatch table.
- Parse Don't Validate shows a newtype. In a statically typed language, a type that only a parse function can construct is the mechanism.

Do not change the prescription to fit the stack. When the stack cannot meet it, the code departs from it (rule 4).

## 3. Never simulate a feature the stack lacks

Do not recommend imitating a language feature the component's language does not have, such as wrapping values in records or classes to simulate branded types in a dynamically typed language. Recommend the closest real mechanism instead (in a dynamic language, a parse function at the boundary that returns checked data), and state the guarantee honestly: the invariant is enforced by convention and run-time checks, not by a compiler.

## 4. Departures, not conformance

When a framework or an established project convention prevents what a lens prescribes, report the code as a departure from that principle, with the constraint as the reason. Examples: ORM entity classes against plain data, a client-side state store against server-rendered state, a dependency-injection container against configuration passed as parameters, mock-based test suites against assertions on pure functions.

- Never describe a departure as conforming, and never restate the principle in weaker terms to make the code pass.
- Use "not applicable" only when the principle's subject is absent from the component, such as browser principles in a component with no browser interface.
- A departure is an architectural finding. `/lenses-fix` reports it and does not apply it automatically.

## 5. Check claims without writing files

A review reads code, documentation, and installed library source to check a claim. It never writes files, inside or outside the workspace, to test one: no scratch scripts, no temporary projects. When a claim can only be confirmed by running code, say so and name the check, such as the test that would confirm it.
