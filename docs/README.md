---
shortDescription: Tiered project documentation — the source of truth for long-lived context beyond .context.md files.
version: 0.1.0
lastUpdated: 2026-03-22
---

## Purpose

`.context.md` files (uses: `skills/context-maintenance.md`) answer "what is in this
directory?" They are low-level, directory-scoped, and maintained automatically by the
Contextualizer. The `docs/` directory answers higher-level questions that no single
directory can: how does the system work, what decisions were made, and what does an agent
need to know before touching a specific domain?

`docs/` is the second tier of context architecture. It is the system of record for
knowledge that is too broad for `.context.md` and too stable for long-term memory.

## Structure

```
docs/
├── README.md          ← this file — index of all docs, load order
├── architecture.md    ← system design, layer boundaries, data flow
├── decisions/         ← architecture decision records (ADRs)
│   └── adr-001-*.md
├── domains/           ← one file per major domain (auth, payments, etc.)
│   └── <domain>.md
└── runbooks/          ← operational procedures (deploy, rollback, debug)
    └── <topic>.md
```

Not all projects need all directories. Start with `architecture.md` and add others as the
project grows. The test for whether something belongs in `docs/` vs `.context.md`: if it
describes relationships, decisions, or flows that span multiple directories, it belongs here.

## File Conventions

Each file in `docs/` MUST follow this structure:

```markdown
# <Title>

**Last updated:** YYYY-MM-DD
**Owned by:** architect | team | [name]

## Summary
[Two to four sentences. What this document covers and why it exists.]

## [Content sections...]

## What Agents Must Know
[Explicit list of constraints, invariants, and non-obvious facts relevant to agents
working in this domain. Formatted as bullets. Actionable, not descriptive.]
```

The `## What Agents Must Know` section is mandatory. It is the distillation that agents
read first — the rest of the document provides context for when they need depth.

## Index

| File | What it covers | When to read it |
|------|---------------|-----------------|
| `architecture.md` | System layers, service boundaries, data flow | Before any cross-layer change |
| `decisions/` | Why specific technical choices were made | When a choice seems wrong or needs revisiting |
| `domains/<name>.md` | Deep context for a specific business domain | Before implementing features in that domain |
| `runbooks/<topic>.md` | Step-by-step operational procedures | Before deploy, rollback, or incident response |

## Maintenance Rules

- Agents MUST update `docs/` when their changes make an existing document inaccurate.
- Agents MUST NOT rewrite a document from scratch — edit only what changed.
- Agents MUST NOT create new docs files without the Architect's approval.
- The `## Last updated` field MUST be updated on every write.
- Documents that have not been updated in 6 months SHOULD be reviewed for staleness.
  Flag them in the handoff under `## Discovered Issues` — do not delete them.
