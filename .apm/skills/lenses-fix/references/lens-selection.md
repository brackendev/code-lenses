# Lens Selection

Part of `/lenses-fix`. These rules choose which lenses run for each component identified by `references/stack-adaptation.md` (section 1). They are this package's guidance. They never name a stack, so they apply to any language, including one nobody planned for.

## The lenses

| Lens | Review skill | What it needs from the code |
|---|---|---|
| grug | `grug-review` | Any code |
| Honest Code | `honest-code-review` | Any code |
| Tidy First? | `tidy-first-review` | Any code |
| Parse Don't Validate | `parse-dont-validate-review` | Data entering from outside the component, such as requests, files, messages, or configuration |
| Data-Oriented Programming | `data-oriented-programming-review` | Application data the component represents and changes |
| APOSD | `aposd-review` | Modules, namespaces, or classes behind interfaces |
| Legacy Code | `legacy-code-review` | Code without tests that the change touches |

## Rules, in order

1. **Explicit requests win.** When the user names lenses, run exactly those (bare names) or add them to the selection (`+name`). A requested lens always runs, even when the rules below would skip it.
2. **Run grug, Honest Code, and Tidy First? for any code.** In a component whose paradigm departs from Honest Code (for example, classes that own mutable state), Honest Code still runs in full, and its paradigm departures are grouped in the report's Architectural departures section.
3. **Run a lens by default when its subject is present and its central idea can apply to the component.** Examples:
   - Parse Don't Validate runs when data enters the component from outside. In a dynamically typed component its central idea still applies through parse functions at the boundary (`references/stack-adaptation.md`, rule 3).
   - Data-Oriented Programming runs when the component already represents its application data with generic structures such as maps and lists.
   - APOSD runs when the component's design is organized around classes and the interfaces between them, and in any component whenever the change alters a public interface or module boundary.
   - Legacy Code runs when the change touches code that has no tests.
4. **Skip a lens by default when the part of it that matters most cannot apply to the component, and the selected lenses already cover the rest.** Example: in a component built on statically typed domain classes, Data-Oriented Programming's central idea (generic data structures) runs against the established representation, and Honest Code P02 and Parse Don't Validate cover its remaining guidance on plain, checked data. A skip is a default, not a judgment of the lens: the report says how to run it.
5. **Owner decisions.** Exceptions to these rules, recorded after an observed bad selection, go here and override rules 2 to 4. There are none yet.

When the evidence for a property is "mixed", decide by what most of the component's code does, and name the minority pattern in the selection reason. For example, a component whose application data is mostly statically typed records, with one field held as a tuple, is built on typed domain data, so rule 4 skips Data-Oriented Programming. When the evidence is "unknown", prefer running the lens and say so in the selection reason.

## Report

Record the selection in the report's Lens selection section: each component with its evidence, each lens run with the rule that selected it, and each lens not run with the reason and the argument that runs it (`+name`).
