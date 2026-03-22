---
shortDescription: Structured handoff schema for machine-readable session output that survives context boundaries.
usedBy: [coder, architect, reviewer, contextualizer]
version: 0.1.0
lastUpdated: 2026-03-22
---

## Purpose

Handoffs written as free prose are difficult for the Maestro and subsequent agents to parse
reliably. When a session ends mid-task or work is re-dispatched across context windows,
structured handoffs become the only reliable source of truth about what happened. This
skill defines a machine-readable handoff schema that every persona uses — consistent enough
for automated parsing, flexible enough to carry the specific information each role needs.

## Schema

All handoffs MUST use the following Markdown structure. Sections marked **required** must
always be present. Sections marked **conditional** are omitted if not applicable.

```markdown
---
persona: coder | architect | reviewer | contextualizer
task: <slug matching the feature spec id or to-do filename>
status: complete | partial | blocked | failed
timestamp: YYYY-MM-DDTHH:MM
---

## Summary
[One sentence. What was accomplished, or why it was not.]

## Deliverables
[Required for coder and architect. List every artifact produced.]
- path/to/file — what it contains and why it changed

## Test Results
[Required for coder. Omit section entirely for other personas.]
- **T1 (Unit):** X passed, 0 failed
- **T2 (Integration):** X passed, 0 failed | not applicable — [reason]
- **T3 (E2E):** X passed, 0 failed | not applicable — [reason]

## Spec Status
[Conditional — include only if a feature spec was attached to the task.]
- ac-1: passing | failing | blocked
- ac-2: passing | failing | blocked

## Decisions
[Conditional — include only if the agent deviated from the plan or brief.]
- [Decision made] — [justification]

## Incomplete
[Conditional — include only if status is partial or blocked.]
- [What was not done] — [reason] — [what the next session must address first]

## Capability Gap
[Conditional — include only if status is failed or blocked by a missing capability.]
[Use the template from skills/error-recovery.md]

## Discovered Issues
[Conditional — include only if pre-existing problems were found outside task scope.]
- [File or area]: [description of issue]

## Session State
[Conditional — include only when ending mid-task so the next session can resume.]
- Last clean commit: <hash>
- WIP commit: <hash> | none
- Failing tests: <test names> | none
- Next step: [exact first action the next session must take]
```

## Frontmatter Fields

| Field | Values | Purpose |
|-------|--------|---------|
| `persona` | coder, architect, reviewer, contextualizer | Who produced the handoff |
| `task` | slug string | Links handoff to spec or to-do — enables programmatic lookup |
| `status` | complete, partial, blocked, failed | Machine-readable outcome |
| `timestamp` | ISO 8601 datetime | When the handoff was produced |

## Status Definitions

| Status | Meaning |
|--------|---------|
| `complete` | All acceptance criteria met, tests passing, clean state |
| `partial` | Some criteria met, work can continue next session, no blockers |
| `blocked` | Cannot proceed — requires human decision, missing capability, or SCOPE error |
| `failed` | Attempted and failed after recovery — Capability Gap section required |

## Rules

- The frontmatter block (between `---` delimiters) MUST appear first, before any Markdown.
- `status: complete` requires: no failing tests, no uncommitted changes, all spec criteria passing (if spec exists).
- `status: partial` requires: a `## Session State` section so the next agent knows exactly where to resume.
- `status: blocked` or `status: failed` requires: an `## Incomplete` section explaining why.
- `status: failed` requires: a `## Capability Gap` section (uses: `skills/error-recovery.md`).
- The `## Summary` section MUST be one sentence. If it takes more than one sentence, the work was not scoped correctly.
- Omit empty sections entirely — do not write `## Decisions\n- none`.

## Guardrails

- Never write a handoff with `status: complete` if tests are failing.
- Never omit the frontmatter. Without it, the Maestro cannot programmatically assess outcome.
- Never use `status: partial` as a way to avoid addressing a blocker. If work is blocked, use `blocked`.
