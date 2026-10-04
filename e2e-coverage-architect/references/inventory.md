# Phase 0 — Inventory

Goal: a written picture of what the app is, how it runs, how auth works, what data exists, and what tests exist today. Write it into `docs/e2e-coverage/matrix.md` under an "Inventory" heading. This section is the foundation for every later phase — a missed route or role here becomes a missed behavior row later, which becomes a permanently untested user flow.

## Steps

### 1. Detect the stack

- Read the project manifest (`package.json`, `pyproject.toml`, `*.csproj`, `go.mod`, …) — framework, scripts, dependencies
- Note the frontend shape (server-rendered, SPA, hybrid) — it determines how routes map to user-visible pages
- If multiple services exist (API + web + worker), note each and which one Playwright should point at

### 2. Find how to run the app

Check in order: existing `playwright.config.*` `webServer`, package scripts (`dev`, `start`), `docker-compose`, README instructions.

- Ambiguous → ask the user one focused question about how to launch the app.
- Cannot be launched in this environment → switch to **analysis-only mode**: complete Phases 1–2 (matrix + gap list), skip Phases 3–4, and state this explicitly in the final report. Do not write tests you cannot run — unrunnable tests are worse than no tests, because they fake coverage.

### 3. Playwright presence

- Locate `playwright.config.*` and existing spec files
- If absent, bootstrap now (see `references/implementation.md` → "Playwright bootstrap"). Report every file you add.
- Run the existing suite once and record the baseline: pass / fail / skip counts. An already-red suite changes triage in Phase 3 — record pre-existing failures in `findings.md` before writing any test of your own.

### 4. Enumerate routes and pages

- Server-rendered / Next.js: filesystem routes (`pages/`, `app/`, `views/`, templates)
- SPA: router configuration (react-router, vue-router, …)
- API: route definitions in server code (routers, controllers, tRPC, GraphQL operations)
- Write the full list. For each page: URL → purpose. For each endpoint: method + path → purpose, auth requirement
- Large repo: you may fan route/test enumeration out to subagents in parallel — but consolidate and verify the merged list yourself. Consistency of the artifacts is your job, not the subagents'.

### 5. Map the auth model

- Mechanism: session cookie, token in storage, OAuth, …
- Roles/permissions: where defined, what each role can do
- Route protection: middleware/guards — and what an unauthenticated or wrong-role request actually receives (redirect? 401? 403?)
- How existing tests authenticate: `storageState`, programmatic login, UI login. You will reuse the project's approach in Phase 3.

### 6. Understand data and persistence

- Storage: database (schema/migrations), files, external service
- Seed data: what exists at boot
- Test isolation story: how state resets between runs. If it doesn't reset, note it — it constrains test design in Phase 3 (create your own data, never assert shared state).

### 7. Existing test inventory

- Spec files, fixtures, helpers, page objects
- One line per spec file: what it appears to cover. Deep per-test analysis happens in Phase 2 — this is just the catalog.

## Exit checklist

Write all of the following into `matrix.md` → "Inventory" before starting Phase 1:

- [ ] Launch command known and verified working (or analysis-only mode declared)
- [ ] Playwright present or bootstrapped; baseline suite result recorded
- [ ] All pages and endpoints listed with auth requirements
- [ ] Auth model and roles documented
- [ ] Data / persistence / seed documented
- [ ] Existing tests cataloged
