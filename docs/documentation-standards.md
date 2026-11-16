# Documentation Standards

Rules for the technical documentation of this project and for the documents that tell AI agents how to work (the "AI specs": `CLAUDE.md`, `docs/*-standards.md`, `.claude/rules/` and `.claude/skills/`).

## General Rules

- Technical documentation, code comments and AI specs are written in **English**.
- Conversation with the user and planning artifacts written for the user are in **Spanish**. `TAREAS.md` stays in Spanish.
- Documentation describes the project as it is. When a document describes something not built yet, it says so and names the card that builds it.
- Cite code as `Class.Member` or by file path. Never cite line numbers: they move with every edit.
- Diagrams are Mermaid blocks inside the Markdown file, so they are versioned and reviewed with the text.
- Keep `CLAUDE.md` under 200 lines. It holds what every session needs; details go to the standards and the skills.

## Technical Documentation

Before every commit, review which documentation the change affects and update it in the same change:

| Change | Update |
|---|---|
| A domain term, aggregate, state or domain event | The glossary and the domain model |
| A table, column, index or constraint | The data model |
| An architectural decision or a new dependency | A new ADR. Do not rewrite an accepted ADR; supersede it |
| A convention, a library, the build or the installation steps | The matching `docs/*-standards.md` and `CLAUDE.md` |
| An endpoint, a webhook or a contract between layers | Its description in the documentation of that area |
| A skill, a plugin or a Claude Code setting | `docs/claude-code-setup.md` and the skills table of `CLAUDE.md` |

When you update documentation:

1. Review the changes made in the code.
2. Identify the documents they affect.
3. Update each one, keeping its structure and style.
4. Report which files you updated and what changed in each.

## AI Specs

The AI specs improve from what happens in the work. When the user corrects an agent, states a preference, or a review finds a mistake that a rule would have prevented, propose a change to the rule that covers it.

- **Ask before you change a rule.** Propose the change and wait for the user's approval.
- **Link the proposal** to the feedback or the finding that motivates it.
- **Be precise.** Name the document and the section, and show the text before and after.
- **Stay in scope.** One piece of feedback changes the rule it concerns, not several unrelated ones.
- **Do not change rules on your own initiative.** A change follows feedback or a finding.
- **Confirm** to the user when an approved change has been applied.
