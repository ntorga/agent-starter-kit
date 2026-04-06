---
shortDescription: Directories that should not be altered or accessed.
scope: coding, planning, review, testing, documentation
version: 0.1.0
lastUpdated: 2026-04-06
---

## Statement

All shell operations (ex: find, ls, etc) MUST NOT include the following directories in the project:
- databases
- rabbitmq_data

## Rationale

Exclude those directories in the project avoid permission errors and changes that could break the project.