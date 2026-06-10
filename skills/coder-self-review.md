---
shortDescription: Deterministic self-evaluation rubric for Coder — scored every run using the GRASP framework.
usedBy: [coder]
version: 0.2.0
lastUpdated: 2026-06-03
---

## Purpose

Before delivering a handoff, the Coder evaluates its own output against the GRASP rubric. Each letter is scored 0, 1, or 2. The total determines whether to deliver, rewrite, or abort. This replaces subjective self-assessment with a deterministic checklist that mirrors the review pipeline — logic before style, conventions before security.

## Procedure

1. **Score each criterion.** After completing the implementation, read the GRASP rubric below and assign a score of 0, 1, or 2 to each letter. Show the scoring breakdown to yourself (internal reasoning, not to the user).

2. **Apply the hard-fail rule.** If any letter scores 0, do not deliver — go to step 3 immediately.

3. **Determine action by total score:**
   - **9 – 10** — **DELIVER** — Implementation meets all criteria. Deliver to user.
   - **7 – 8** — **FIX the scored < 2 criteria.**
     a. Identify which letters scored below 2.
     b. Fix those gaps automatically (do NOT consult the user).
     c. Re-score, then deliver if 9-10.
     d. If still below 9-10, retry once more.
     e. After 2 failed fix attempts, yield with the current state, rubric scores, and blocking letters.
   - **0 – 6** — **RESTART** — The implementation is fundamentally broken. Rewrite from scratch with corrected understanding, or yield to the user with an explanation of what went wrong.

## GRASP Rubric

### G — GUIDELINES

_Did I follow the playbook end-to-end with no scope creep and clear technical reasoning?_

- **0** — Skipped todo creation or management. Did not review context files before touching code. Tests do not pass or were not run. Handoff format is wrong, missing sections, or absent. Scope expanded beyond the plan/brief.
- **1** — Todo managed and context reviewed, but one or more procedural gaps: did not load relevant skills before implementing, did not consult the Google Style Guide for the language, did not update `.context.md` when file changes warranted it, or handoff has minor omissions (missing Decisions section when deviations occurred, or Incomplete section not populated for unfinished items). Technical reasoning in handoff or commit message is vague.
- **2** — All playbook steps followed: existing todo checked (or new one created), plan type determined, context files reviewed, neighboring files read for style, **Google Style Guide for the language consulted and followed**, relevant skills loaded, test-first approach used (tests failed before implementation), all tests pass, `.context.md` updated where changes warranted, acceptance criteria verified, handoff delivered in exact format with all sections populated. **Handoff and commit message clearly explain technical reasoning and impact.** Yield conditions evaluated honestly — yielded when warranted, continued when appropriate.

### R — REASONING

_Will this code survive real-world input and error paths?_

- **0** — Logic does not match the task brief's acceptance criteria. Error paths silently swallowed. Boundary conditions (nil, empty, zero, off-by-one) not handled. Resource leaks in error paths (unclosed connections, file handles, channels). Incomplete work markers present (TODO, FIXME, stub returns, skipped tests without justification).
- **1** — Logic is sound and tests pass, but one or more edge cases are uncertain: retry logic missing backoff, external calls lack timeouts, backward compatibility of API changes not verified, or one error path logs but does not propagate.
- **2** — All error paths handled or explicitly logged. Boundary conditions tested. No resource leaks. All external calls have timeouts. No incomplete markers. Backward compatibility verified or breaking changes documented in handoff Decisions. Tests cover Good, Bad, and Ugly lenses per the plan.

### A — ARCHITECTURE

_Does this code respect the project's architectural boundaries?_

- **0** — Change violates layer boundaries (outer layer depends on inner, or vice versa). New dependency flows against the project's dependency direction. Duplicated logic where extraction exists, or premature abstraction where a simple function would do.
- **1** — Boundaries respected, but one concern is unclear: a new cross-cutting dependency might create a cycle at scale, or an abstraction's necessity is questionable (not obviously wrong, but not obviously needed either).
- **2** — Layer boundaries respected per `.context.md` definitions. All dependencies flow in the correct direction. No duplication — existing utilities reused where applicable. No premature abstractions — code is as simple as the problem requires. Structural coherence check passes.

### S — STYLE

_Did this code look like it belongs in this codebase and follow the Google Style Guide for the language?_

- **0** — Code does not match the local style of surrounding files (different naming conventions, structure, or patterns). Violates the Google Style Guide for the language. Commandment-level rule violations present. Cryptic one-liners or clever patterns that need comments to understand. Unjustified lint/type suppression markers added.
- **1** — Style mostly matches, but one or two naming or formatting inconsistencies exist against the surrounding code or the Google Style Guide. Counsel-level deviations present without visible justification. One lint suppression added without an adjacent comment explaining why.
- **2** — Read two neighboring files before writing and matched their style exactly. **Follows Google Style Guide conventions for the language** (naming, imports, formatting, docstrings). All naming follows project conventions. Code is readable without comments — the structure explains itself. No new lint suppressions, or each has a clear adjacent justification. All `code-` rules checked and followed. Commandments respected, edicts justified, counsel noted.

### P — PROTECTION

_Did I map and secure every point where untrusted data enters?_

- **0** — Change introduces an endpoint, handler, or data flow accepting external input. Auth not enforced on mutating operations. Secrets visible in source or config. No sanitization on data reaching SQL, templates, file paths, or command sinks.
- **1** — Attack surface identified and basic sanitization present, but one area is uncertain: auth enforcement relies on middleware ordering convention rather than explicit attachment, error messages may leak internals, or rate limiting missing on auth-adjacent endpoints.
- **2** — No new attack surface (score 2 immediately). If surface exists: every untrusted data flow traced to its sink, parameterized queries used, auth explicitly attached to each endpoint, no secrets in source, TLS validation intact, crypto uses modern algorithms with adequate key lengths.

## Guardrails

- Never deliver if any letter scores 0 — regardless of total. A zero is a hard fail.
- Never skip scoring any letter — all 5 must be evaluated every run.
- The rubric is fixed — do not add or remove criteria. If a criterion proves inadequate, file a framework change request.
- When fixing gaps (score 7-8 range), only address the letters that scored below 2. Do not rework letters that already scored 2. Fix automatically — do NOT stop to consult the user.
- After 2 failed fix attempts, yield — do not keep looping. Present the current state, rubric scores, and blocking letters to the user.
- The "restart" action (score 0-6) means: do not deliver the current output. Rewrite the implementation from scratch with corrected understanding, or yield to the user with a clear explanation of the failure mode.
