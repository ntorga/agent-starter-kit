---
shortDescription: Three-tier testing protocol — unit, integration, and E2E browser validation before any handoff.
usedBy: [coder, reviewer]
relatedTo: [playwright, vitest, jest, pytest]
version: 0.1.0
lastUpdated: 2026-03-22
---

## Purpose

Agents that ship code without running tests produce false confidence. A feature that "looks right" in the context window but fails in a browser is not done. This skill defines a three-tier testing protocol that enforces verification at every level — from isolated logic to full browser-based E2E — before any handoff leaves the Coder persona.

## Tiers

| Tier | Scope | Primary Tool | Required When |
|------|-------|--------------|---------------|
| **T1 Unit** | Isolated function or module | Project's existing runner (vitest, jest, pytest, go test) | Every code change |
| **T2 Integration** | API endpoints, DB queries, service boundaries | Project's existing runner + test fixtures | Feature touches service or data layer |
| **T3 E2E** | Full user flow via headless browser | Playwright CLI (`npx playwright test`) | Feature has UI, or acceptance criteria describe user behavior |

## Procedure

1. **Detect the test stack.** Before writing any tests, scan for the project's existing test configuration:

   ```bash
   ls vitest.config.* jest.config.* pytest.ini pyproject.toml playwright.config.* go.mod 2>/dev/null
   ```

   If no test runner is configured, flag this in the handoff as a **blocker** under `## Discovered Issues`. Do not install a test runner without the Architect's approval — it is an architectural decision.

2. **Run the existing suite first (smoke check).** Establish a baseline before touching any code:

   ```bash
   # Adapt to the project's runner
   npx vitest run --reporter=verbose 2>&1 | tail -30
   # or
   npx playwright test --reporter=list 2>&1 | tail -30
   ```

   If pre-existing tests fail, document them in `## Discovered Issues` in the handoff. Do not fix them unless they directly block the current task's acceptance criteria.

3. **Write T1 unit tests for new or changed logic.** Every function with branching logic, error paths, or non-trivial computation MUST have a unit test. Test names MUST describe the behavior being verified, not the function:

   ```
   ✗ test("processOrder")
   ✓ test("rejects order when inventory is insufficient")
   ✓ test("applies discount only to eligible items")
   ```

4. **Write T2 integration tests when crossing service boundaries.** If the feature touches an API endpoint, database query, message queue, or external service, write integration tests using test fixtures or a test database. Mock only what you cannot control (third-party APIs, payment gateways). Do not mock your own services.

5. **Write and run T3 E2E tests for user-facing features.** Use Playwright CLI as the primary tool — it runs headless by default, produces disk-based output, and is CI-native:

   ```bash
   # Run all E2E tests headless
   npx playwright test

   # Run a specific spec file
   npx playwright test tests/e2e/checkout.spec.ts

   # Run with trace capture for debugging
   npx playwright test --trace on

   # View trace after a failure
   npx playwright show-trace test-results/<test-name>/trace.zip
   ```

   E2E tests MUST simulate real user behavior — test as a human would:
   - Navigate to the entry point (do not deep-link past authentication)
   - Fill forms with realistic, representative data
   - Assert visible outcomes (text on screen, URL change, element presence/absence)
   - Cover error states (invalid input, empty states, network errors)

   **MCP mode (optional, for investigation only):** If the project has the Playwright MCP server configured, the Coder MAY use it to interactively explore visual bugs or debug a failing E2E scenario. MCP is for investigation — test execution always uses CLI.

   ```json
   {
     "mcpServers": {
       "playwright": {
         "command": "npx",
         "args": ["@playwright/mcp@latest", "--headless"]
       }
     }
   }
   ```

6. **Run the full suite and verify green.** After all changes are in place:

   ```bash
   npx vitest run && npx playwright test
   ```

   All tests MUST pass before the handoff is written. If a test fails, apply the error recovery protocol (uses: `skills/error-recovery.md`). Fix the code — not the test assertion — unless the test expectation is demonstrably wrong.

7. **Report results in the handoff.** Include a `## Test Results` section:

   ```markdown
   ## Test Results
   - **T1 (Unit):** 47 passed, 0 failed
   - **T2 (Integration):** 12 passed, 0 failed
   - **T3 (E2E):** 8 passed, 0 failed
   - **Coverage delta:** +3.2% (if coverage tooling is configured)
   ```

   If a tier was skipped (e.g., no UI = no T3), state why explicitly.

## Guardrails

- Never mark a feature as passing without completing all applicable tiers.
- Never skip the smoke check (step 2). Pre-existing failures must be documented, not hidden.
- Never write tests that pass vacuously — no empty assertions, no `expect(true).toBe(true)`.
- Never mock your own services or database in integration tests. Mocks at service boundaries hide real bugs.
- Never install a test framework without explicit architectural approval.
- Never use MCP for test execution. MCP is for exploration, CLI is for verification.
