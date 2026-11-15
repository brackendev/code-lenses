# Lens Selection

Part of `/lenses-fix`. These rules choose which lenses run for each component identified by `references/stack-adaptation.md` (section 1). They are this package's guidance. They never name a stack, so they apply to any language, including one nobody planned for.

## The code under review

The rules judge the code under review, which depends on the scope:

- For the default scope (changed files), it is the changed lines and the smallest named function or top-level definition that contains each of them. A statement that sits directly inside a function that wraps a whole file counts as a top-level definition. A change to a comment or a docstring puts only those lines under review. Deleted code is not under review, but read it to judge whether the change removes a public function, a dispatch, or a dependency.
- For a path or a glob, it is every file in scope. For `all`, it is every file that the skill sampled, or every file when the codebase is small enough to read in full.

A subject that exists only elsewhere in the component does not select a lens. For the default scope, rule 3 selects a lens only when a changed line contains its subject. The unchanged lines of the containing function are context for the review, not a reason to select a lens. For example, a component that reads requests in one file does not select Parse Don't Validate for a change to a function that only formats a value.

These rules narrow the selection for the default scope. A path, a glob, or `all` asks for a review of everything in scope, so most lenses run there. A change that adds a whole feature can also select every lens.

The component properties in `references/stack-adaptation.md` describe the whole component, so read as much of the component as they need. The selection reads only the code under review.

## Terms

- **Component** always means a component as `references/stack-adaptation.md` defines it.
- **Module** means a class, a namespace, a package, or a script file with its own public names. Use the unit in which the language's convention groups public names: usually a class in a class-based language, and a namespace or a file elsewhere. Only the project's own modules count, not the standard library or third-party packages.
- **Outside data** is data that enters the running program from somewhere else: a request, a response from a server, a file, a message, a row read from a database, configuration, an environment variable, a command-line argument, the output of another process (not its exit status), or browser storage. A browser script and the server it talks to are separate programs. Data passed between modules of one program is not outside data. Code interprets outside data when it uses the content of a value: it decodes it, checks it, branches on it, computes with it, or stores it. Selecting, removing, or renaming entries by key and passing their values on does not interpret them.
- **Application data** is the data about the problem the program solves, such as orders, users, or work items, which the program stores, sends, or displays. Option maps, argument maps, and values local to one function are not application data.
- **State that outlives one call** is a value that the program changes and reads again later: a mutable variable or field, a global, a cache, or a row or file that the code writes. A value that stays fixed while the process runs, such as an environment variable, is not state, and neither is a mutable object that one call creates and discards.
- **A dispatch over variants** is a branch, a table, or a polymorphic call that chooses behavior by the kind of a value, where more than two kinds exist or can be added. A two-way branch on a flag is not one.
- **A tested function** has at least one test that checks the result of its main path, directly or through a caller. A test that reaches only a failure return does not count.

## The lenses

| Lens | Review skill | What it needs from the code under review |
|---|---|---|
| grug | `grug-review` | Any code |
| Honest Code | `honest-code-review` | Any code |
| Tidy First? | `tidy-first-review` | Any code |
| Parse Don't Validate | `parse-dont-validate-review` | Code that interprets outside data. In a statically typed component, also a type that represents application data |
| Data-Oriented Programming | `data-oriented-programming-review` | Application data held in generic structures. For the default scope, the change must add a kind of application data, or add, rename, or remove a field of one |
| Simple Made Easy | `simple-made-easy-review` | State that outlives one call, or a dispatch over variants. For a path, a glob, or `all`, also modules that depend on each other |
| APOSD | `aposd-review` | For the default scope, a change to a public interface or to the set of modules. Otherwise, two or more modules, one of which uses another's public names |
| Legacy Code | `legacy-code-review` | A function that is not tested |

## Rules, in order

1. **Explicit requests win.** When the user names lenses, run exactly those (bare names) or add them to the selection (`+name`). A requested lens always runs, for every component, even when the rules below would skip it.
2. **Run grug, Honest Code, and Tidy First? for any code.** Each has a subject that any code can contain:
   - grug is the only lens that argues against added structure. Every other lens can recommend more parts, types, or schemas. When grug disagrees with one of them, the report lists the disagreement under Conflicts and the finding is not applied automatically. Keep grug in the selection even when a selected lens is likely to disagree with it.
   - Tidy First? finds small structural tidyings, which any code can need.
   - Honest Code covers faults, defaults, and side effects, which any code can have. In a component whose paradigm departs from Honest Code (for example, classes that own mutable state), Honest Code still runs in full, and its paradigm departures are grouped in the report's Architectural Departures section.

   Prose documentation, and a change that touches only comments, is not code. No lens runs for it, and the report names those files in one line below the Lens Selection table. Build and dependency files are evidence for the component properties, not code under review.
3. **Run each other lens when the code under review contains its subject, unless rule 4 skips it.** Test code selects no lens under this rule. It is reviewed with the code it tests, by the lenses which that code selects. Test support code, such as fixtures and a test launcher, is test code.
   - Parse Don't Validate runs when the code under review interprets outside data. The code that reads the data can be unchanged code elsewhere. Code that forwards outside data without using the content of its values does not select it. In a statically typed component, the lens also runs when the code under review defines a type that represents application data, because the lens covers the states such a type allows. For the default scope, the change must add or alter that type. In a dynamically typed component its central idea still applies through parse functions at the boundary (`references/stack-adaptation.md`, rule 3).
   - Data-Oriented Programming runs in a component that represents its application data with generic structures such as maps and lists. For the default scope, the change must also add a kind of application data, or add, rename, or remove a field of one. An added option key or a single small return value does not select it. A typed collection of entity classes is not a generic structure.
   - Simple Made Easy runs when the code under review creates, reads, or changes state that outlives one call, or contains a dispatch over variants. For the default scope, a dispatch selects it only when the change adds, removes, or alters it. For a path, a glob, or `all`, it also runs when the scope holds modules that depend on each other. A change that only adds or removes a dependency between modules does not select it for the default scope. Code that only computes from its arguments does not select it.
   - APOSD runs for the default scope when the change adds or removes a module, or adds, removes, or renames a public function, method, or interface, or changes its parameters or the options it accepts. For a path, a glob, or `all`, it runs when the scope holds two or more modules and one uses another's public functions or interfaces, whatever the component's paradigm.
   - Legacy Code runs when a function under review is not tested. Judge this by reading the tests, without running a coverage tool. A test added in the same change counts. An untested branch in a tested function does not select the lens. Name the untested functions in the selection reason.
4. **Skip a lens by default when the part of it that matters most cannot apply to the component, and the selected lenses already cover the rest.** Example: in a component built on statically typed domain classes, Data-Oriented Programming's central idea (generic data structures) runs against the established representation, and Honest Code P02 and Parse Don't Validate cover its remaining guidance on plain, checked data. A second example: in a component where classes own mutable state, Simple Made Easy's ideas of values over state and functions over methods run against the established paradigm, and Honest Code (P02, P03, P04) reports the same designs and the same departures. A skip is a default, not a judgment of the lens: the report says how to run it.
5. **Owner decisions.** Exceptions to these rules, recorded after an observed bad selection, go here and override rules 2 to 4. There are none yet.

When the evidence for a component property is "mixed", decide by what most of the component's code does, and name the minority pattern in the selection reason. For example, a component whose application data is mostly statically typed records, with one field held as a tuple, is built on typed domain data, so rule 4 skips Data-Oriented Programming. When the evidence is "unknown", prefer running the lens and say so in the selection reason. A subject that is absent is "none", not "unknown", and selects nothing.

## What each lens receives

For the default scope, a lens that rule 3 selected receives the files whose code selected it, every other file in scope that calls that code or that it calls, and the test files in scope that exercise those files. Legacy Code receives the files that hold an untested function and the test files that exercise them, for any scope. The rule 2 lenses receive every code file in scope.

## Report

Record the selection in the report's Lens Selection section: each component with its evidence, each lens run with the rule that selected it, and each lens not run with the reason (`rule 3: subject absent`, or rule 4) and the argument that runs it (`+name`). For a lens that rule 3 selected, name the subject and give the file reference of the code that selected it.
