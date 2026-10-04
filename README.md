# e2e-coverage-architect

A Claude skill that turns spotty E2E testing into systematic behavior coverage.

Point it at an existing web app and it will:

1. **Inventory** the app — stack, launch command, routes, API endpoints, auth model, data, and every existing test
2. **Enumerate behaviors** — user-action units ("regular user deletes own post", "admin views user list") across six dimensions: happy path, validation, authorization, error handling, state transitions, persistence
3. **Analyze gaps** — map existing tests onto a behavior-coverage matrix with per-cell evidence (`file:line`), and detect not just missing tests but **duplicates** and **false greens** (tests that pass without actually asserting behavior)
4. **Implement** prioritized Playwright tests (P0 auth/data-loss → P1 core flows → P2 secondary), running each batch and triaging failures
5. **Loop** — re-analyze until no meaningful gaps remain (max 3 passes, honest reporting either way)

Completion is judged by **behavior coverage** — never by test count or route count.

## Install

```bash
npx skills add <owner>/e2e-coverage-architect
```

## What it touches

- Test code and test configuration (including bootstrapping Playwright if absent)
- `docs/e2e-coverage/` analysis artifacts (`matrix.md`, `gaps.md`, `findings.md`) in the target project

**It never modifies application source code.** If a new test exposes an app bug, the bug is recorded as a known-failure in `findings.md`, the test is marked `test.fixme` with a reference, and the app is left alone. What you do with the findings is your call.

## Requirements

- A web app with a browser frontend that can run locally (the skill needs to launch it)
- Playwright — or nothing: the skill bootstraps it if missing

## Example

> Our e2e tests are thin and I'm not sure what's actually covered. Figure out what user flows are untested and add Playwright tests for the important stuff.
