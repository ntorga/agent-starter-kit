---
shortDescription: Universal code quality conventions for all languages.
scope: coding
version: 0.1.0
lastUpdated: 2026-03-04
---

## Statement

### Variable Naming

Variable names MUST convey intention or purpose, not describe content. Single-letter variable names MUST NOT be used.

### Error Handling

Errors MUST always be logged. An error that passes without a log entry is a silent failure. Handling depends on context — if the error is non-critical, it may be logged and not propagated, but it must never be swallowed silently.

### Testing

Complex logic MUST have unit tests. Test error messages MUST be descriptive and provide context. Secrets, certificates, and private keys MUST NOT be hardcoded in tests.

### Comments

Comments SHOULD be treated as a code smell. If a block needs a comment to be understood, review the logic first — the code itself may need to be clearer. Comments are acceptable only when the logic cannot speak for itself.

## Rationale

These conventions produce code that reads linearly, names that communicate intent, and tests that explain failures. They reduce cognitive load during review and make codebases navigable by both humans and agents who arrive without prior context.
