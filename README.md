# Agent Starter Kit

A structural foundation for building multi-agent AI systems. Provides personas, rules, skills, and an orchestration pattern you can adapt to any project.

## Setup

1. Clone this repo into a `.agents/` directory in the project you want to work on.
2. Add `.agents/` and `.memory/` to the project's `.gitignore`.
3. Copy the `CLAUDE.md` file from this repo into the root of the project (or point your AI tool's entry file to `.agents/README.md`).
4. Start the AI agent (e.g., `claude`, or whatever CLI you use).
5. Ask the agent to run the **Contextualizer** first — it will walk the codebase and produce `.context.md` orientation files that all other personas rely on.
6. From there, speak naturally. The Maestro orchestrates everything.

## Structure

```
personas/    Specialized AI roles (who does the work)
rules/       Constraints organized by authority level
skills/      Reusable procedures and protocols
```

## How It Works

The **Maestro** is the conductor. It receives user requests, decomposes them, and dispatches work to specialized personas:

- **Architect** — plans implementations, defines before/after states
- **Coder** — writes software following the plan
- **Reviewer** — checks work for correctness and quality
- **Contextualizer** — documents project structure for orientation

Each persona has an identity (who they are), a playbook (what they do), a handoff format (what they deliver), and red lines (what they must not do).

## Rules Hierarchy

- **Commandments** — absolute, never bypassed
- **Edicts** — authoritative within scope, not bent
- **Counsel** — wise guidance, may be deviated from with justification

## Skills

Skills codify procedures that personas reference. They answer "how to do X" so personas can focus on "what to do."

- **boot** — session startup sequence
- **dispatch** — how the Maestro assembles and sends work to personas
- **task-tracking** — file-based to-do for multi-step work

## Customization

- Add new personas to `personas/` following the schema in `personas/README.md`
- Add rules to `rules/commandments/`, `rules/edicts/`, or `rules/counsel/`
- Add skills to `skills/` following the schema in `skills/README.md`
- Modify existing files to match your project's needs

Each directory has a README with the full schema definition.
