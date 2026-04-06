# Rules

Rules are constraints — short, direct, and non-procedural. A rule that needs multiple pages to explain is likely a skill in disguise.

## Rule Hierarchy

- **Commandments** (`rules/commandments/`) — sacred, absolute, never bypassed.
- **Edicts** (`rules/edicts/`) — authoritative within their scope, shall not be bent.
- **Counsel** (`rules/counsel/`) — wise guidance, may be deviated from with justification.

## Available Rules

| Rule                                 | Description                                                       | Scope                                            |
|--------------------------------------|-------------------------------------------------------------------|--------------------------------------------------|
| `commandments/git`                   | Conventional commits, branch naming, commit style                 | coding                                           |
| `commandments/forbidden-directories` | Skipped directories to avoid permission errors                    | coding, planning, review, testing, documentation |
| `edicts/code-quality`                | Universal naming, testing, and quality conventions                | coding                                           |
| `edicts/code-debugging`              | Root cause before fix, three-strike rule, anti-rationalization    | coding                                           |
| `counsel/clarification`              | When ambiguity warrants stopping vs proceeding with best judgment | communication                                    |

## File Naming

Lowercase, hyphenated. Scoped rules are prefixed with the persona or domain they target: `coder-formatting.md`, not `formatting.md`. Universal rules carry no prefix.

## Schema (v0.1.0 // 2026-03-04)

### Frontmatter

| Field              | Required | Explain                                 | Example                                              |
| ------------------ | -------- | --------------------------------------- | ---------------------------------------------------- |
| `shortDescription` | Yes      | What the rule enforces in one sentence. | `Mandates .context.md updates on structural changes` |
| `scope`            | Yes      | Task category this rule applies to.     | `coding`                                             |
| `version`          | Yes      | Semantic version.                       | `0.1.0`                                              |
| `lastUpdated`      | Yes      | Last modification date.                 | `2026-02-05`                                         |

### Body

| Section   | Required | Purpose                                                                                                               |
| --------- | -------- | --------------------------------------------------------------------------------------------------------------------- |
| Statement | Yes      | The rule itself. Use RFC-style language: MUST, MUST NOT, SHOULD, SHALL, SHALL NOT. As short as the constraint allows. |
| Rationale | Yes      | Why this rule exists. One paragraph. Without rationale, rules feel arbitrary and get ignored.                         |
