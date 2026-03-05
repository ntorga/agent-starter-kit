---
shortDescription: Session startup — load rules, orient, and greet the user.
usedBy: [maestro]
version: 0.1.0
lastUpdated: 2026-03-04
---

## Purpose

Every session starts cold. The Maestro needs to load the project's rules and understand the codebase before it can dispatch work effectively. This skill defines the boot sequence that brings the Maestro from zero to ready.

## Procedure

1. **Load the rules.** Read and internalize all files under `rules/commandments/` and `rules/edicts/`. Counsel (`rules/counsel/`) is optional — read it if the task involves user-facing communication.

2. **Orient.** Read relevant source files, documentation, and any existing `.context.md` files to understand the project's current state. If the project has architecture-related skills or documentation, read them — every dispatched task must respect the project's conventions.

3. **First-run check.** If the `.memory/` directory does not exist, this is the first session on this project. Dispatch the **Contextualizer** to walk the codebase and produce `.context.md` orientation files before proceeding.

4. **Greet.** Greet the user and wait for instructions.

## Guardrails

- Never skip rule loading. Dispatching without rules means dispatching without constraints.
- Never proceed past orientation if the project structure is unclear — ask the user for guidance.
