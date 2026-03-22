---
shortDescription: Classify errors, attempt recovery, and escalate with a capability gap report — never retry blindly.
usedBy: [coder, maestro]
version: 0.1.0
lastUpdated: 2026-03-22
---

## Purpose

Agents that retry blindly waste tokens and context window without converging on a solution. This skill defines a structured approach: classify the error, attempt recovery with a different strategy, and if recovery fails, escalate with a precise description of the missing capability. The goal is to turn every unresolved failure into a legible, enforceable improvement to the harness — not a vague "it didn't work."

## Error Classes

| Class | Description | Example |
|-------|-------------|---------|
| **ENV** | Environment not ready | Missing dependency, wrong runtime version, port already in use |
| **LOGIC** | Code bug in new changes | Test failure, type error, off-by-one, null reference |
| **UPSTREAM** | Pre-existing bug exposed by changes | Legacy test fails, undocumented API behavior |
| **TOOL** | Agent's tool failed transiently | CLI timeout, file permission denied, API rate limit |
| **SCOPE** | Task requires work outside the brief | Needs infra change, design decision, or credentials the agent lacks |
| **UNKNOWN** | Cannot classify | Cryptic error, no stacktrace, ambiguous output |

## Procedure

1. **Capture the error in full.** When a command or test fails, read the complete error output — do not truncate. Store the raw message for use in step 4 if escalation is needed.

2. **Classify.** Match the error against the table above. If ambiguous, classify as UNKNOWN and proceed.

3. **Attempt recovery — maximum 2 attempts per error.** Apply the strategy for the class:

   - **ENV** — run the project's setup script or install the missing dependency:
     ```bash
     [ -f init.sh ] && bash init.sh
     ```
     If a specific package is missing, install it and re-run.

   - **LOGIC** — read the failing test's error message carefully. Identify the root cause in the code. Fix the code (not the test assertion, unless the expectation itself is wrong). Re-run only the failing test:
     ```bash
     npx vitest run path/to/failing.test.ts
     # or
     npx playwright test tests/e2e/failing.spec.ts
     ```

   - **UPSTREAM** — do not fix. Document in `## Discovered Issues` in the handoff. If the upstream bug directly blocks the current task, implement a minimal workaround and document the trade-off.

   - **TOOL** — wait 5 seconds and retry once. If still failing, escalate.

   - **SCOPE** or **UNKNOWN** — do not attempt recovery. Proceed directly to step 4.

4. **Escalate if both attempts fail.** Stop attempting and include a **Capability Gap** section in the handoff:

   ```markdown
   ## Capability Gap

   **Error class:** ENV | LOGIC | UPSTREAM | TOOL | SCOPE | UNKNOWN
   **Error (verbatim):**
   > [exact error output — maximum 20 lines]

   **Attempts:**
   1. [what you tried] → [result]
   2. [what you tried] → [result]

   **Missing capability:** [what the harness, environment, or agent needs to handle this properly]
   **Suggested fix:** [concrete action — new skill, new tool, config change, or human decision required]
   ```

5. **Continue with remaining work.** A single blocked item does not halt the entire task. Mark the blocked item `[-]` in the to-do with a brief reason, and continue with all unblocked items.

6. **Maestro handles capability gaps.** When the Maestro receives a handoff containing a `## Capability Gap` section, it MUST present the gap to the user and propose one of three options:
   - **Fix now** — dispatch a new sub-agent to address the missing capability, then re-dispatch the original task.
   - **Defer** — add the gap to `.memory/long-term.md` under `## Discovered Issues` for a future sprint.
   - **Cancel** — abandon the blocked work item and update the feature spec status to `blocked`.

## Guardrails

- Never retry the same approach more than twice. The third attempt is always escalation.
- Never reframe a failure as "try harder" — the fix is always a missing capability in the harness.
- Never swallow errors silently. Every error must be classified and either resolved or reported.
- Never fix UPSTREAM bugs unless they directly block the current task's acceptance criteria.
- Never attempt SCOPE errors — yield immediately. Scope changes are the Maestro's decision.
