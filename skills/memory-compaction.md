---
shortDescription: Prune stale entries and distill long-term memory before it degrades session quality.
usedBy: [maestro]
version: 0.1.0
lastUpdated: 2026-03-22
---

## Purpose

Long-term memory (`skills/agent-memory.md`) grows append-only. Over weeks and months on
an active project, `.memory/long-term.md` accumulates contradictory preferences, resolved
issues still listed as open, and lessons that have since been codified into rules. A bloated
memory file degrades every session that loads it — agents spend context window on noise,
and outdated entries actively mislead.

This skill defines when and how to compact memory: removing stale entries, resolving
contradictions, and promoting stable lessons into the rules hierarchy where they belong.

## When to Run

Compact memory when any of the following is true:

- `.memory/long-term.md` exceeds 150 lines
- The framework has been updated (run automatically during boot after a pull — uses: `skills/boot.md`)
- More than 4 weeks have passed since the last compaction (check `## Last Compacted` header)
- A persona reports that a memory entry directly contradicts current behavior

Do not run during an active task. Compaction changes the memory file — run it as a
standalone session step before dispatching other work.

## Procedure

1. **Read the full memory file.** Load `.memory/long-term.md` and the current
   `rules/commandments/` and `rules/edicts/` to cross-reference.

2. **Remove resolved entries.** Delete any entry under `## Discovered Issues` where the
   issue has since been fixed (verify against recent git log):

   ```bash
   git log --oneline --since="90 days ago" | grep -i "<issue-keyword>"
   ```

   If a fix commit exists, the issue is resolved — remove the entry.

3. **Remove superseded entries.** Delete any entry under `## Learned Rules` or
   `## Preferences` that is now covered by an explicit rule in `rules/`:

   ```bash
   grep -r "<entry-keyword>" rules/
   ```

   If the behavior is already enforced by a commandment or edict, the memory entry is
   redundant — remove it.

4. **Resolve contradictions.** Scan for entries in the same section that conflict
   (e.g., "always use tabs" and "always use spaces"). Keep the most recent entry.
   Remove the older one. Log the resolution:

   ```markdown
   - YYYY-MM-DD: Removed contradicting preference "X" — superseded by "Y" (added YYYY-MM-DD)
   ```

5. **Promote stable lessons.** If a `## Learned Rules` entry has survived at least
   two compaction cycles without being removed, it is stable. Propose promoting it to
   a formal rule:

   - If it is project-specific and narrow → add to `rules/counsel/`
   - If it is broadly applicable and important → propose to user as a new edict

   Do not create rules without surfacing the promotion to the Maestro for user confirmation.

6. **Deduplicate.** Scan each section for entries expressing the same idea in different
   words. Keep the clearer, more specific one.

7. **Update the header.** Add or update the `## Last Compacted` line at the top of
   `.memory/long-term.md`:

   ```markdown
   ## Last Compacted
   YYYY-MM-DD — removed N entries, resolved N contradictions, promoted N to rules
   ```

8. **Commit the compacted memory.** Memory files are in `.gitignore` by default — do not
   commit them. If this project tracks memory in git, commit with:

   ```bash
   git add .memory/long-term.md
   git commit -m "chore(memory): compact — remove stale entries, resolve contradictions"
   ```

## Guardrails

- Never remove an entry without verifying it is actually resolved, superseded, or contradicted.
  When in doubt, keep it.
- Never promote a lesson to a formal rule without surfacing it to the user. Rules change agent
  behavior globally — that is a human decision.
- Never run compaction during an active implementation task. It changes the context other
  agents depend on mid-flight.
- Never compact memory that was written less than 7 days ago. Fresh entries have not been
  validated by real usage.
