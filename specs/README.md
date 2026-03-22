---
shortDescription: JSON feature specifications with per-criterion status tracking.
version: 0.1.0
lastUpdated: 2026-03-22
---

## Purpose

Feature specs are the contract between the Architect and the Coder. They define what "done" looks like in machine-readable JSON — not prose. Each spec has a `status` field that agents update as they work, and each acceptance criterion has its own individual status. This creates a clear, session-persistent record of progress that no agent needs to infer from context.

Using JSON (not Markdown) reduces the risk of agents inadvertently editing the spec during implementation. A spec file is a read-mostly artifact — only the `status`, `updated`, and `notes` fields should change after creation.

## Schema

Each spec lives at `specs/<prefix>-<slug>.json`, where `<prefix>` follows the conventional-commit convention (`feat`, `fix`, `refactor`) and `<slug>` is kebab-case.

```json
{
  "id": "feat-user-auth",
  "title": "User Authentication with OAuth2",
  "status": "failing",
  "priority": 1,
  "phase": 1,
  "totalPhases": 3,
  "created": "YYYY-MM-DD",
  "updated": "YYYY-MM-DD",
  "riskLevel": "high",
  "description": "One paragraph. What the feature does and why it matters.",
  "acceptanceCriteria": [
    {
      "id": "ac-1",
      "description": "Testable, observable behavior — written from the user's perspective.",
      "status": "failing"
    }
  ],
  "affectedFiles": [
    "src/auth/oauth.ts",
    "tests/e2e/auth.spec.ts"
  ],
  "notes": ""
}
```

## Field Reference

| Field | Who sets it | Who updates it | Notes |
|-------|------------|----------------|-------|
| `id` | Architect | — | Immutable after creation |
| `title` | Architect | — | Immutable after creation |
| `status` | Architect | Coder, Reviewer | Derived from `acceptanceCriteria` statuses |
| `priority` | Architect / Maestro | Maestro | Lower number = higher priority |
| `phase` / `totalPhases` | Architect | — | Current phase of multi-phase implementation |
| `created` / `updated` | Architect | Any writer | `updated` changes on every write |
| `riskLevel` | Architect | — | `low`, `medium`, `high` — maps to approval gate |
| `description` | Architect | — | Immutable after creation |
| `acceptanceCriteria[].status` | Architect (sets `failing`) | Coder | Updated to `passing` when test passes |
| `affectedFiles` | Architect | Coder | Coder adds files discovered during implementation |
| `notes` | Any persona | Any persona | Blockers, decisions, deviations |

## Status Values

| Value | Meaning |
|-------|---------|
| `failing` | Not yet implemented, or test exists and fails |
| `in-progress` | Coder is actively working on this spec |
| `passing` | All acceptance criteria verified green by tests |
| `blocked` | Cannot proceed — reason in `notes` |

The top-level `status` reflects the aggregate of its criteria:
- All criteria `passing` → spec `passing`
- Any criterion `failing` or `in-progress` → spec `in-progress` or `failing`
- Any criterion `blocked` with no path forward → spec `blocked`

## Risk Level → Approval Gate

| `riskLevel` | Gate required |
|-------------|---------------|
| `low` | None — Reviewer pass is sufficient |
| `medium` | Reviewer `pass` verdict required before merge |
| `high` | Explicit user approval required (Maestro surfaces `[DESTRUCTIVE]` tag) |

## Rules

- Specs MUST be created by the Architect. The Coder reads specs — it does not create or rewrite them.
- Acceptance criteria MUST be individually testable and observable. "Works correctly" is not a valid criterion.
- The `description`, `acceptanceCriteria[].description`, and `id` fields are immutable after creation. If requirements change, the Maestro creates a new version of the spec.
- The Coder MUST update `acceptanceCriteria[].status` from `failing` to `passing` only after the corresponding test passes in CI (not just locally).
- Agents MUST NOT delete spec files. Completed specs are archived by moving them to `specs/archive/`.
