---
shortDescription: File-based to-do tracking for multi-step and multi-session work.
usedBy: [coder]
version: 0.2.0
lastUpdated: 2026-03-22
---

## Purpose

Agents working on multi-step tasks need a way to track progress that survives session boundaries. A task that spans hours or days will be picked up by an agent with no memory of what was already done. This skill defines a file-based to-do format that any persona can create, update, and resume — so the next session starts where the last one stopped.

## Procedure

1. **Check for an existing to-do.** Before starting work, look for files in the `.memory/todo/` directory at the project root. If a to-do file for the current task already exists, read it and resume from the first unchecked item. Do not recreate.

2. **Create a to-do from the plan, task brief, or feature spec.** If no existing to-do matches, create `.memory/todo/YYYY-MM-DD-<prefix>-<slug>.md` following the schema below.

   **Naming convention:** The filename uses the pattern `YYYY-MM-DD-<prefix>-<slug>.md`, where `<prefix>` is the conventional-commit type and `<slug>` is a short kebab-case summary (follows: `rules/commandments/git.md`). Examples: `.memory/todo/2026-02-18-feat-user-auth.md`, `.memory/todo/2026-02-18-fix-login-redirect.md`.

   **When a feature spec exists** (`specs/<id>.json`), derive to-do items directly from its `acceptanceCriteria` array. Each criterion becomes one checkbox item, referencing the spec and criterion ID:

   ```markdown
   - [ ] [feat-user-auth / ac-1] User can click 'Sign in with Google' and complete OAuth flow
   - [ ] [feat-user-auth / ac-2] User can click 'Sign in with GitHub' and complete OAuth flow
   ```

   When marking an item `[x]`, also update the corresponding `acceptanceCriteria[].status` in the spec JSON from `"failing"` to `"passing"`. If no spec exists, derive items from the architect plan's phases and file lists or from the task brief's acceptance criteria.

3. **Update as you go.** After completing each item, mark it `[x]` and add a log entry if the outcome was notable (unexpected decision, deviation from plan, blocker encountered). Do not batch updates — mark items done as they finish.

4. **Handle blockers.** If an item cannot be completed, mark it `[-]` with a reason in the log. Continue with unblocked items. If nothing can proceed, stop and note the blocker in the log.

5. **Close the to-do.** When all items are done (or skipped with reason), delete the to-do file. The handoff summary is the permanent record — the to-do is a working document, not an archive.

## Schema

```markdown
# <Task Title>

**Slug:** <prefix>-<topic>
**Source:** plan | task-brief
**Created:** YYYY-MM-DD
**Updated:** YYYY-MM-DD

## Items

- [x] Completed item
- [ ] Pending item
- [-] Skipped item

## Log

- YYYY-MM-DD: Notable event, decision, or blocker.
```

**Schema notes:**

- **Slug** matches the `<prefix>-<topic>` portion of the filename and follows the git branch prefix convention.
- **Source** identifies whether items came from an architect plan or a task brief.
- **Updated** changes every time an item is checked off or a log entry is added.
- Items use standard checkbox syntax: `[x]` done, `[ ]` pending, `[-]` skipped.
- Log entries are chronological. Keep them terse — one line per event.

## Guardrails

- Never start implementing without first creating or locating an existing to-do.
- Never let the to-do drift from reality. If you completed something, mark it now, not later.
- Never delete a to-do with unchecked items unless the task has been explicitly cancelled.
