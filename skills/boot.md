---
shortDescription: Session startup — gitignore, auto-update, memory, rules, context, and greet.
usedBy: [maestro]
version: 0.4.0
lastUpdated: 2026-03-22
---

## Purpose

Every session starts cold. The Maestro needs to ensure the project is wired correctly, the framework is up to date, load the project's rules, and understand the codebase before it can dispatch work effectively. This skill defines the boot sequence that brings the Maestro from zero to ready.

## Procedure

1. **Gitignore.** Ensure `.agents/` and `.memory/` are in the project's `.gitignore`. Run:

   ```bash
   touch .gitignore
   grep -qxF '.agents/' .gitignore || echo '.agents/' >> .gitignore
   grep -qxF '.memory/' .gitignore || echo '.memory/' >> .gitignore
   ```

2. **Framework pull.** Run:

   ```bash
   git -C .agents pull
   ```

   - If the pull brought changes:
     - Read the `CHANGELOG.md` in `.agents` to understand what changed.
     - Purge obsolete long-term memory entries — compare `.memory/long-term.md` against the changelog and remove any entry that the update now handles natively.
     - Stop and reboot — re-read the Maestro persona from the top so updated instructions take effect.
   - If already up to date, continue.

3. **Memory.** Load long-term memory (uses: `skills/agent-memory.md`). If `.memory/long-term.md` exceeds 150 lines or the framework was just updated in step 2, run memory compaction before proceeding (uses: `skills/memory-compaction.md`).

4. **Load the rules.** Read and internalize all files under `rules/commandments/` and `rules/edicts/`. The `rules/commandments/security.md` file has scope `all` — it applies to every task regardless of type. Counsel (`rules/counsel/`) is optional — read it if the task involves user-facing communication.

5. **Docs.** If a `docs/` directory exists at the project root, read `docs/README.md` to understand the project's knowledge base structure. Read any domain files relevant to the current task before dispatching.

6. **Context.** Verify the project has context files. Run:

   ```bash
   find . -name ".context.md" -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/vendor/*" -not -path "*/.cache/*" -print -quit
   ```

   - If `find` produces no output, no `.context.md` files exist. Dispatch the **Contextualizer** (uses: `personas/contextualizer.md`) before proceeding.

7. **Greet.** Greet the user and wait for instructions.

## Guardrails

- Never skip rule loading. Dispatching without rules means dispatching without constraints.
- Never skip the framework pull. An outdated `.agents` directory means outdated instructions.
