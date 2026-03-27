# Changelog

```log
0.3.2 - 2026/03/27
feat(dispatch): add providers codex and cursor to routing matrix

0.3.1 - 2026/03/26
fix(agents,boot): symlink-aware paths — AGENTS.md references use .agents/ prefix, boot skill hints bare paths resolve under .agents/

0.3.0 - 2026/03/26
refactor: merge README boot content into AGENTS.md — single-hop boot
feat(skills): add loop-recovery — structured retry pivot/abandon with oscillation and drift detection
feat(skills): add code-sec-review — OWASP-aligned security code review checklist
feat(skills): add code-coherence-review — logic, correctness, and structural integrity checks
feat(skills): add code-quality-review — rules-walk procedure for coding standards compliance
feat(rules): add code-debugging edict — root cause before fix, three-strike rule, anti-rationalization
feat(rules): add clarification counsel — stop/proceed/escalate taxonomy, clarify-plan-act gates
feat(personas): rewrite reviewer — lean identity with three structured passes (coherence, quality, security), prompt injection red line
feat(rules): expand code-quality edict — data trust boundary, process-killing exceptions, richer testing, schema change disclosure
refactor(boot): load rules index only — Maestro reads rules/README.md, sub-agents read individual rules when dispatched
refactor(dispatch): expand rule loading from commandments-only to all tiers, add loop-recovery to dispatch notes

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
