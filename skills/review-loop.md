---
shortDescription: LOC-based review tier selection with shapeshifter dispatch for the unified reviewer.
usedBy: [maestro]
version: 0.1.0
lastUpdated: 2026-04-07
---

## Purpose

Different scales of change need different review depth. This skill measures the scope of code changes, splits oversized scopes into reviewable blocks, selects the appropriate review tier, and dispatches the reviewer with the right focus instructions. It uses a single unified reviewer (`personas/reviewer.md`) dispatched multiple times with different cognitive focus per tier.

## Procedure

1. **Measure scope.** For code changes, count the lines changed:
   ```bash
   git diff --numstat | awk '{ s += $1 + $2 } END { print s }'
   ```
   For plans and non-code work, skip to step 3 and use Unified tier.

2. **Split if needed.** If total LOC exceeds 1500, dispatch the Contextualizer in review scoping mode (uses: `personas/contextualizer.md`, dispatch via: `skills/dispatch.md`) to group changed files into blocks of 1500 or fewer LOC. Each block proceeds independently through step 3.

3. **Select tier and dispatch** (dispatch via: `skills/dispatch.md`). Each dispatch uses the same `personas/reviewer.md` persona. The task brief overrides the reviewer's focus — the reviewer shapeshifts into the role described in the brief while following the same playbook.
   - **Unified** (< 500 LOC) — single dispatch of `personas/reviewer.md`.
   - **Standard** (500–1000 LOC) — two dispatches of `personas/reviewer.md`, each with a `<review-focus>` block in the task brief:
     - `<review-focus>Coherence and quality: trace logic paths, verify completeness, check naming and style against loaded rules.</review-focus>`
     - `<review-focus>Security: trace untrusted inputs to dangerous sinks, check auth boundaries, verify error handling.</review-focus>`
   - **Full** (1000–1500 LOC) — three dispatches of `personas/reviewer.md`, each with a `<review-focus>` block in the task brief:
     - `<review-focus>Coherence: trace logic paths, verify completeness, check structural integrity.</review-focus>`
     - `<review-focus>Quality: check naming, style, and patterns against loaded rules.</review-focus>`
     - `<review-focus>Security: trace untrusted inputs to dangerous sinks, check auth boundaries.</review-focus>`
     Suggest the user consider an external review tool (e.g., CodeRabbit, Greptile) for additional coverage at this scale.

4. **Merge findings.** When multiple dispatches run, merge findings: union all blockers, warnings, and notes; deduplicate identical entries. Conflicting verdicts resolve to the stricter one.

5. **Verify findings.** Spot-check each blocker and warning against the codebase before acting. Reviewers can hallucinate — discard false positives (invented violations, misread paths, fabricated rules). Only confirmed findings proceed.

6. **Act on verdict.**
   - `pass` — proceed to delivery.
   - `partial-pass` — no blockers but a review step was skipped. Surface the gap to the user.
   - `fail` — present the verified findings to the user, incorporate any additional input, re-dispatch the Coder with the findings, re-review. Repeat until the verdict is `pass` or `partial-pass`.

## Guardrails

- Never skip the verify step. Reviewers — especially when dispatched to cheaper models — can hallucinate findings. Every blocker and warning must be confirmed against the actual codebase before acting on it.
