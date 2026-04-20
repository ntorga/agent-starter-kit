# Skills

Skills are collected intelligence on how to operate a specific tool — whether that is a CLI, an API, or an MCP/ACP server. They codify procedures, protocols, and output formats that personas reference during execution.

### Available Skills

| Skill                   | Description                                                                                 |
| ----------------------- | ------------------------------------------------------------------------------------------- |
| `agent-memory`          | Long-term and session memory across sessions                                                |
| `boot`                  | Session startup — gitignore, auto-update, memory, rules, orient                             |
| `code-coherence-review` | Logic coherence, correctness, and structural integrity checks                               |
| `code-quality-review`   | Rules-walk procedure for coding standards compliance                                        |
| `code-sec-review`       | OWASP-aligned security code review checklist                                                |
| `context-maintenance`   | How to maintain .context.md files as the project evolves                                    |
| `dispatch`              | Assembles sub-agent prompts with task brief                                                 |
| `loop-recovery`         | Structured recovery and escalation for retry loops                                          |
| `plan-critique`         | Adversarial plan validation — structural checks and assumption attack before implementation |
| `reviewer-handoff`      | Structured review summary format with verdict logic and deterministic coverage scoring       |
| `reviewer-scoring`  | Deterministic checklist-based review scoring — replaces subjective confidence                |
| `review-loop`       | LOC-based review tier selection with shapeshifter dispatch for the unified reviewer          |
| `task-tracking`         | File-based to-do tracking for multi-step and multi-session work                             |

## When to Extract a Skill

Extract a skill when:

- A tool proves difficult enough that a human must step in and write an explicit how-to for the agent to follow.
- A procedure must be standardized across multiple personas, such as a shared protocol or output format.

Do not extract when the procedure is short and intuitive. If a competent agent can work it out without written guidance, a skill file adds overhead without value.

## File Naming

Lowercase, hyphenated: `task-tracking.md`, `dispatch.md`

Persona-specific skills are prefixed with the persona name: `coder-linting.md`, `reviewer-checklist.md`. Universal skills carry no prefix.

## Schema (v0.1.0 // 2026-03-04)

### Frontmatter

| Field              | Required | Explain                                                                  | Example                                      |
| ------------------ | -------- | ------------------------------------------------------------------------ | -------------------------------------------- |
| `shortDescription` | Yes      | What the skill does in one sentence.                                     | `Cross-session memory retrieval and storage` |
| `usedBy`           | Yes      | Which personas use this skill. `[all]` if injected universally via boot. | `[all]` or `[maestro]`                       |
| `relatedTo`        | No       | External tools, CLIs, or APIs this skill wraps or abstracts.             | `[docker, awk]` or `[anthropic-api]`         |
| `version`          | Yes      | Semantic version.                                                        | `0.1.0`                                      |
| `lastUpdated`      | Yes      | Last modification date.                                                  | `2026-02-05`                                 |

### Body

| Section    | Required | Purpose                                                                                                                                                                                                                               |
| ---------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Purpose    | Yes      | What this skill does and why it exists. One paragraph, no bullet points. Answer "what problem does this solve?" not "what steps does it take."                                                                                        |
| Procedure  | Yes      | Numbered steps for executing the skill. Each step that produces an artifact must describe its output inline — format, structure, and destination. Reference other skills or rules with `(uses: path)` or `(follows: path)` as needed. |
| Guardrails | No       | Skill-specific pitfalls to avoid. Not commandments, not procedure repetition. Ask: "what mistake would an agent make when using this skill carelessly?"                                                                               |
