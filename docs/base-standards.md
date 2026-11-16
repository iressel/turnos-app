# Base Standards

Rules that apply to every change in this repository, whoever writes it: a person or an AI agent. The area documents refine them: [backend-standards.md](./backend-standards.md) and [documentation-standards.md](./documentation-standards.md).

## 1. Core Principles

- **Small steps, one at a time.** Work in small increments and finish one before starting the next.
- **Test-driven development.** Start new behaviour with a failing test.
- **Type safety.** All code is fully typed. Nullable reference types stay enabled and warnings are errors.
- **Clear naming.** Names say what a thing is or does. Prefer a longer exact name to a short vague one.
- **Incremental changes.** Prefer a focused change to a large one. A change does one thing.
- **Question assumptions.** State the assumptions a change depends on and check the ones that can be checked.
- **Pattern detection.** Point out repeated code and repeated decisions, and propose the abstraction only when the repetition is real.
- **Immutability by default.** Treat data as immutable unless mutation is necessary. Value objects and DTOs are immutable.
- **Chesterton's fence.** Before you refactor or delete code, find out why it exists and what depends on it, and say so.
- **Dependency agnosticism.** Reach an external library or service through an interface of ours (a port), so it can be replaced and faked in tests.
- **Early return.** Check the negative conditions first and return, so the main flow stays flat.

## 2. Language

- **English** for every technical artifact: code, comments, error and log messages, test names, technical documentation, configuration, scripts, database schema and commit messages.
- **Spanish** for conversation with the user, planning artifacts written for the user, `TAREAS.md`, and every text an end user reads (es-AR).
- **One name per domain concept.** The glossary maps each Spanish domain term to its English code name. Use that name in every identifier and comment, and add a term to the glossary before you use it in code.
- **Name things after what they do**, not after the first customer, provider or screen that needed them. A provider's name belongs only in the adapter that talks to that provider.

## 3. SOLID and Clean Code

- **Single responsibility.** A class has one reason to change. If describing it needs the word "and", split it.
- **Open/closed.** Add behaviour with a new class or strategy instead of growing an `if` or `switch` that already branches on a type or a mode.
- **Liskov substitution.** A subtype honours the whole contract of its base type. No `NotImplementedException` in a subtype.
- **Interface segregation.** Prefer small role interfaces to a large general one.
- **Dependency inversion.** Depend on abstractions. `Domain` and `Application` define the ports; `Infrastructure` implements them.
- **No empty `catch`.** Never swallow an error. Handle it, or let it reach the layer that logs it and tells the user.
- **No dead code.** Delete unused code instead of commenting it out; git keeps the history.
- **Comments explain why**, not what. Public members get XML documentation.

## 4. Changes to Existing Code

- **Match the surrounding code.** Follow the pattern the area already uses. Introducing a new framework or architectural style is a decision of its own, recorded in an ADR, never a side effect of a feature.
- **Keep the layout of a file you edit.** Do not reformat lines the change does not need. A whole-file reformat hides the real change in the diff and conflicts with every open branch.
- **Known debt is not a pattern.** A defect that already exists is not a reason to repeat it. Fix it when the change touches it and it is in scope; otherwise record it.

## 5. Self-Check Before Delivering

Before you present a change as done, check:

1. Does it keep the separation of concerns: rules in `Domain`, orchestration in `Application`, technology in `Infrastructure`, presentation in `Web`?
2. Does the UI use the design tokens (colours, spacing, typography) instead of literal values?
3. Does the code follow the SOLID principles above?
4. Did the tests run, and did you read their output?

If an answer is no, fix it before you respond. Report what you verified and what you could not verify.
