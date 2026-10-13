---
name: boot
description: Session startup — gitignore, auto-update, memory, rules, context, CLI config, and greet.
usedBy: [maestro]
version: 0.5.1
lastUpdated: 2026-10-13
---

## Purpose

Every session starts cold. The Maestro needs to ensure the project is wired correctly, the framework is up to date, load the project's rules, and understand the codebase before it can dispatch work effectively. This skill defines the boot sequence that brings the Maestro from zero to ready.

## Path Convention

All framework files live under `.agents/`. Markdown references within the framework use bare paths for readability — always resolve them under `.agents/`. Shell commands always use the `.agents/` prefix for project-root paths.

## Procedure

1. **Gitignore.** Ensure `.agents/`, `.memory/`, and `opencode.json` are in the project's `.gitignore` — except where the project already tracks them on purpose. Run:

   ```bash
   touch .gitignore
   for entry in '.agents/' '.memory/' 'opencode.json' '.ignore'; do
       [ -n "$(git ls-files "$entry" | head -1)" ] && continue
       grep -qxF "$entry" .gitignore || echo "$entry" >> .gitignore
   done
   ```

   The `git ls-files` guard exists because some projects vendor the framework as a git
   subtree, so `.agents/` is tracked deliberately. Ignoring an already-tracked path does not
   untrack what is there, so nothing breaks the day the line is added — but every file added
   to the framework afterwards becomes invisible to git, and the subtree starts losing content
   silently. Skipping an entry that already has tracked files is always correct: a path the
   project tracks is a path the project decided to track.

2. **Framework pull.** Check how `.agents/` is vendored:

   ```bash
   [ -e .agents/.git ] && echo clone || echo vendored
   ```

   - **`clone`** — `.agents/` is its own repository. Run `git -C .agents pull`.
   - **`vendored`** — `.agents/` is a git subtree, or plain files, committed into the host project. Do not run `git -C .agents pull`. With no `.git` of its own, git resolves `.agents` to the host repository, so the command pulls the host project instead of the framework, and fails whenever the host branch has diverged from its remote. The host project owns the framework sync, for example a `git subtree pull --prefix=.agents <framework-remote> main --squash` in its own update script. If the project documents such a script and it has not run this session, run it. Otherwise tell the user the framework was not updated, and continue.

   - If the pull brought changes:
     - Read the `CHANGELOG.md` in `.agents` to understand what changed.
     - Purge obsolete long-term memory entries — read `.memory/long-term.md`, read the changelog, and for each memory entry remove it only if the changelog describes a feature, skill, or rule that replaces that memory's purpose. If the entry's purpose is not clearly covered by the changelog, keep it.
     - Re-read `personas/maestro.md` from the top so updated instructions take effect.
   - If already up to date, continue.

3. **Memory.** Load memory (uses: `skills/agent-memory/SKILL.md`).

4. **Core skills.** Read the following core skills IN FULL now. These are mandatory for every dispatch and planning decision. Do not skip, summarize, or rely on memory:
   - `skills/dispatch/SKILL.md` — required for every sub-agent dispatch
   - `skills/plan-management/SKILL.md` — required for planning decisions and epic tracking

5. **CLI configuration.** Run:

   ```bash
   bash .agents/skills/assets/maestro-boot-configure-cli.sh
   ```

    - If the script reports the config was `created` or `updated`, inform the user that agent bindings changed and they should restart the session for the new bindings to take effect.
    - If the script reports `unchanged`, no action is needed.
    - If `yq` or `jq` is not installed, the script prints a skip message — no action needed.
    - If no supported CLI config file is found, the script exits silently — no action needed.

6. **Load the rules index.** Read `rules/README.md` to know what rules are available and their scopes. Do not read the individual rule files — sub-agents will read them when dispatched.

7. **Context.** Verify the project has context files. Run:

   ```bash
   find . -name ".context.md" -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/vendor/*" -not -path "*/.cache/*" -print -quit
   ```

    - If `find` produces no output, no `.context.md` files exist. Dispatch the **Contextualizer** (uses: `personas/contextualizer.md`) before proceeding.
    - Also check `docs/FEATURE-MAP.md` — if it is missing, the Contextualizer dispatch covers it (uses: `skills/context-maintenance/SKILL.md`).

8. **Greet.** Greet the user and wait for instructions. Remind the user: they are not talking to a single agent — they are talking to a team of specialists that can handle multiple requests simultaneously, so large and complex prompts are welcome.

## Guardrails

- Never skip rule loading. Dispatching without rules means dispatching without constraints.
- Never skip the framework pull. An outdated `.agents` directory means outdated instructions.
