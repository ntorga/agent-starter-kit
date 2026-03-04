---
shortDescription: Git workflow rules for all coders.
scope: coding
version: 0.1.0
lastUpdated: 2026-03-04
---

## Statement

All commits MUST use conventional commit prefixes: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`.

Branch names MUST follow the same prefixes: `feat-*`, `fix-*`, `refactor-*`, `docs-*`, `test-*`, `chore-*`.

Commit messages MUST be short — a single phrase. If you need to describe too much, you should have committed earlier.

## Rationale

Consistent commit conventions enable automated changelogs, semantic versioning, and make git history readable. Short commits with clear intent are easier to review, revert, and bisect.
