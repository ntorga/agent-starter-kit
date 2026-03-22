---
shortDescription: Checkpoint strategy, clean state definition, and revert playbook for agent-driven work.
usedBy: [coder, maestro]
version: 0.1.0
lastUpdated: 2026-03-22
---

## Purpose

Agents working across long sessions or multiple context windows can leave the codebase in
an ambiguous state — partial implementations, passing tests on broken foundations, or work
that cannot be safely merged. This skill defines three things: what "clean state" means,
how to create checkpoints during long work, and how to recover when something goes wrong.
It makes git the source of truth for progress — not the agent's memory.

## Clean State Definition

A branch is in **clean state** when all of the following are true:

- All tests pass (`npx vitest run && npx playwright test` exit 0)
- No uncommitted changes (`git status` shows nothing to commit)
- No debug code, commented-out blocks, or TODO comments introduced by the current task
- All acceptance criteria in the feature spec are `passing` (if a spec exists)
- No unresolved merge conflicts
- The branch can be merged to the target branch without breaking CI

Clean state is the only acceptable condition for a handoff. Partial work must be committed
to a checkpoint branch — not left as uncommitted changes.

## Procedure

### Creating Checkpoints

1. **Before starting a phase.** Create a checkpoint commit immediately before making
   any changes, capturing the known-good baseline:

   ```bash
   git add -A && git commit -m "chore: checkpoint before <phase-name>"
   ```

2. **After each logical unit of work.** Do not batch changes across multiple features or
   files into a single commit. Each commit must be a working increment:

   ```bash
   # Stage only the files changed in this unit
   git add path/to/changed/files
   git commit -m "<type>: <description>"
   ```

3. **Before a risky operation.** Tag the current state before any operation that modifies
   many files, changes dependencies, or touches configuration:

   ```bash
   git tag checkpoint/<slug>-<YYYYMMDD>
   ```

   List existing checkpoints:
   ```bash
   git tag -l "checkpoint/*"
   ```

### Recovering from a Bad State

When tests fail after a series of changes and the root cause is unclear, recover in this
order — stop at the first step that restores clean state:

1. **Stash and re-test.** Stash uncommitted changes and verify the last commit is green:

   ```bash
   git stash
   npx vitest run && npx playwright test
   ```

   If green → the uncommitted changes introduced the problem. Inspect the stash:
   ```bash
   git stash show -p
   ```
   Apply selectively, re-run tests after each file.

   If still failing → the problem is in the committed history. Proceed to step 2.

2. **Bisect to the bad commit.** Find the last green commit and bisect:

   ```bash
   git log --oneline -10        # find a known-good commit hash
   git bisect start
   git bisect bad               # current state is bad
   git bisect good <hash>       # known-good commit
   # git bisect runs automatically; run tests after each step
   npx vitest run && npx playwright test
   git bisect good              # or: git bisect bad
   ```

   When bisect identifies the culprit commit, inspect it:
   ```bash
   git show <culprit-hash>
   git bisect reset
   ```

3. **Revert the bad commit.** If the culprit is a discrete, revertible commit:

   ```bash
   git revert <culprit-hash> --no-edit
   npx vitest run && npx playwright test
   ```

4. **Reset to checkpoint.** If revert is not clean, reset to the nearest checkpoint tag:

   ```bash
   git reset --hard checkpoint/<slug>-<YYYYMMDD>
   ```

   Flag this as `[DESTRUCTIVE]` — any uncommitted work after the checkpoint is lost.
   Report to Maestro before executing.

5. **Escalate.** If none of the above restores clean state, stop. Produce a Capability
   Gap report (uses: `skills/error-recovery.md`) describing what state the codebase is
   in and what specific recovery action is needed from a human.

### Session Handoff via Git

When a session ends before a task is complete, ensure the next session can resume without
guessing:

1. Commit all in-progress work — even if tests are failing — with a clear prefix:

   ```bash
   git add -A && git commit -m "wip: <what is done> — tests failing on <what>"
   ```

2. Leave a `## Session State` section in the to-do file (uses: `skills/task-tracking.md`):

   ```markdown
   ## Session State
   - Last clean commit: <hash>
   - WIP commit: <hash>
   - Failing tests: <test names>
   - Next step: <exactly what the next session must do first>
   ```

3. The next session MUST read this section before touching any code.

## Guardrails

- Never leave uncommitted changes at session end. Commit or stash — never abandon.
- Never reset to a checkpoint without flagging `[DESTRUCTIVE]` to the Maestro first.
- Never use `git push --force` on a shared branch. Flag it as `[DESTRUCTIVE]` and wait for explicit user approval.
- Never bisect without first stashing. Running tests on a dirty working tree produces unreliable results.
