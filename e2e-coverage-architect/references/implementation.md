# Phase 3 — Implementing tests

## Order and batching

Work gaps in priority order (P0 → P1 → P2). Implement in small batches — one behavior-cluster at a time (e.g., "everything around post deletion") — and run the batch before moving on. Small batches keep triage cheap: when something fails, you know exactly which change caused it.

## Conventions first

Before writing the first test, study the existing specs and copy their patterns: fixtures, page objects, locator style, test-data strategy, naming. New tests should read like the project wrote them. If no tests exist, keep it simple: plain spec files, a shared auth helper, behavior-based test titles ("regular user cannot delete another user's post").

## Authentication in tests

- Reuse the project's existing approach (`storageState`, programmatic login) when present
- Otherwise prefer a programmatic login helper (POST to the login endpoint, reuse the session) over walking the UI — faster and less brittle
- Walk the UI login form only when login itself is the behavior under test

## Test isolation

- Each test creates its own data (via API or UI) and asserts on what it created
- Use unique identifiers (timestamp / random suffix) so parallel workers never collide
- Never depend on execution order or another test's leftovers
- If the project has no reset strategy: create-don't-reuse, and clean up via API where possible

## Assertion quality

Assert user-visible outcomes: content on screen, URL — and, for the persistence dimension, state after a reload or re-fetch. A persistence assertion without a reload is a happy-path assertion in disguise.

## Running and triage

Run each batch (e.g. `npx playwright test tests/e2e/auth.spec.js`). For each failure:

| Symptom | Meaning | Action |
|---|---|---|
| locator/wait timeout, selector mismatch | test-side | fix the test |
| setup data missing or duplicated | test-side | fix isolation |
| assertion fails: app returns wrong content/status for correct input | **app-side bug** | known-failure flow below |
| failure already present in the Phase 0 baseline | app-side, pre-existing | record in `findings.md` if not already there; don't block on it |

### Known-failure flow (app-side bugs)

1. Write a `findings.md` entry: `findings#N — <title>`, repro steps, the failing assertion, and evidence (what the app does vs. what it should do)
2. Mark the test as fixme with a reason referencing the entry, e.g. `test.fixme(true, 'known failure: findings#N — edit drops title changes')` (or the project's equivalent)
3. Keep the test body intact — it documents the bug and goes live the moment the app is fixed
4. Continue with the next gap

Never edit application source to make a test pass. Never delete or weaken a failing test to make a suite green.

If a bug blocks an entire area (e.g., login broken), record it once, fixme the dependent tests, and continue elsewhere. A blocked area is not a finished pass — say so in the report.

## Playwright bootstrap (when absent)

Minimal setup; report every file you add:

- `npm i -D @playwright/test` and `npx playwright install chromium` (or the project's preferred browser set)
- `playwright.config.ts` with `webServer` wired to the launch command found in Phase 0, a `baseURL`, one browser to start
- `tests/e2e/` directory
- `.gitignore` entries for `test-results/` and `playwright-report/`

Bootstrap is test infrastructure — it is not an app-source change.
