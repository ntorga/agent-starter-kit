---
shortDescription: Deterministic checklist-based review scoring — replaces subjective confidence.
usedBy: [reviewer]
version: 0.1.0
lastUpdated: 2026-04-07
---

## Purpose

Subjective confidence scores are unreliable — research shows most models inflate self-assessment to 4–5 out of 5 regardless of actual coverage. This skill replaces subjective judgment with a deterministic checklist. Each item is binary (done or not done). The score is computed, not judged.

## Procedure

1. **Complete the coverage checklist.** After finishing the review, answer each item with `[x]` (done) or `[ ]` (not done). Do not skip items — unanswered items count as not done.

   ```markdown
   ### Coverage

   - [ ] Read every changed file in full (not just the diff summary)
   - [ ] Traced all entry points to their exit points in the changed code
   - [ ] Checked each changed function/method against loaded rules
   - [ ] Verified error handling paths in the changed code
   - [ ] Checked interactions between changed code and its immediate callers/callees
   - [ ] Verified that no finding was reported without evidence (file path + line or logic trace)
   ```

2. **Compute the score.** Count checked items. Report as `**Coverage: N/6**` where N is the count of checked items.

3. **Flag gaps.** For each unchecked item, add a one-line explanation of why it was not completed (e.g., "could not trace callers — no call site found in changed files"). These gaps inform Maestro whether to re-dispatch or accept.

## Guardrails

- Never check an item you did not actually do. An honest 3/6 is more useful than a fabricated 6/6.
- Never add or remove checklist items. The checklist is fixed — it measures coverage, not findings.
