# Changelog

```log
0.4.0 - 2026/03/22
feat: add git-recovery skill — checkpoint strategy, clean state definition, bisect/revert/reset playbook, session handoff via WIP commits
feat: add memory-compaction skill — prune resolved issues, remove superseded rules, resolve contradictions, promote stable lessons to rules/
feat: add handoff-format skill — machine-readable frontmatter (persona, task, status, timestamp) with structured sections for all personas
feat: add docs/ directory — tiered context layer for architecture, decisions, domains, and runbooks with mandatory "What Agents Must Know" section
feat(boot): add docs/ tier to boot sequence; trigger memory compaction when .memory/long-term.md exceeds 150 lines or framework updates
feat(coder): add git-recovery checkpoint step to Playbook; delegate handoff structure to handoff-format skill
feat(dispatch): always include git-recovery and handoff-format for all Coder and persona dispatches
refactor: remove evals, observability, and HITL from Sprint 2 scope — redundant for CLI-based usage (Claude Code + OpenCode)

0.3.0 - 2026/03/22
feat: add testing skill — three-tier protocol (unit, integration, E2E) with Playwright CLI as primary tool and MCP as optional investigation mode
feat: add security commandment (scope: all) — input validation, least-privilege, secret protection, output sanitization, destructive action gates, dependency verification (OWASP Agentic Top 10)
feat: add error-recovery skill — 6 error classes, max-2-retry rule, Capability Gap report template, Maestro escalation flow
feat: add specs/ directory and README — JSON feature spec schema with per-criterion status tracking (failing → passing)
feat(architect): produce JSON feature spec alongside plan document; set riskLevel for high-impact changes
feat(coder): reference testing and error-recovery skills in Playbook; add Test Results section to handoff; update spec criterion status as tests pass
feat(maestro): pass spec path in task briefs; handle Capability Gap escalations (fix/defer/cancel); surface [DESTRUCTIVE] actions for user approval
feat(reviewer): cross-reference spec acceptanceCriteria against handoff Test Results; flag mismatches as blockers
feat(task-tracking): derive to-do items from specs/*.json acceptanceCriteria; sync item completion back to spec status
feat(boot): note security.md scope:all applies to every task
feat(dispatch): always include testing and error-recovery skills for Coder dispatches

0.2.3 - 2026/03/10
feat(boot): add gitignore check — ensure .agents/ and .memory/ are in project .gitignore on startup
feat(boot): add auto-update — pull latest framework on session start, reboot if changes detected
feat(boot): add long-term memory loading and inline memory purge after framework update
feat(boot): replace vague Orient/First-run steps with deterministic context check via find
feat: add agent-memory skill — long-term memory across sessions
feat: add context-maintenance skill — .context.md schema and update rules
refactor(contextualizer): reference context-maintenance skill for .context.md schema
docs(readme): replace vague setup steps with deterministic shell commands (git clone, ln -s)
docs(readme): consolidate FAQ — merge Why This, Why Multi-Model, and Best Practices into FAQ section
docs(readme): add FAQ entries for auto-update, model assignment, and model-agnostic explanation with ELO reference
docs(readme): clean intro — fix tagline comma splice, drop fillers, replace "ultra-personalized" with "fully customizable"

0.2.2 - 2026/03/09
feat(maestro): surface pre-existing issues (bugs, tech debt, code smells) found by sub-agents during Deliver
feat(dispatch): instruct sub-agents to report pre-existing issues in a Discovered Issues handoff section
fix(maestro): strengthen commit guardrail — add Red Line, require unambiguous "commit" authorization (not just approval)

0.2.1 - 2026/03/06
refactor: rename CLAUDE.md to AGENTS.md for model-agnostic branding
docs(readme): merge intro and Motivation into two concise paragraphs, add provider examples and ELO requirement
refactor(maestro): update Identity — chief of staff role, accountability, clarification over prohibition
feat(maestro): add commit flow to Handoff — branch guard, user confirmation, git commandment reference

0.2.0 - 2026/03/05
feat(dispatch): add multi-provider routing with Providers table, host runtime detection, and native/CLI dispatch logic
feat(personas): add preferredModel frontmatter field for provider routing
refactor(dispatch): split overloaded step 1 into discrete steps (identify runtime, extract fields, select provider, decide dispatch mode)

0.1.0 - 2026/03/04
feat: initial release
feat: add 5 personas (maestro, architect, coder, reviewer, contextualizer)
feat: add dispatch skill with native subagent prompt assembly
feat: add boot skill for session startup
feat: add task-tracking skill for file-based to-do
feat: add git commandment and code-quality edict
feat: add schema definitions for personas, rules, and skills
```
