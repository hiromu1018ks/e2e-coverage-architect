---
name: e2e-coverage-architect
description: Analyze an existing web app's routes, auth, permissions, data flows, and current tests to build a behavior-coverage matrix; detect coverage gaps, duplicate tests, and false-green tests; then implement prioritized Playwright E2E tests and loop until meaningful behavior gaps are exhausted. Use this skill whenever the user wants to improve, audit, extend, or distrust-check E2E/test coverage, add missing Playwright tests, find untested user flows, verify that existing E2E tests actually assert behavior, or set up Playwright coverage from scratch — even if they don't say "coverage" explicitly.
---

# E2E Coverage Architect

Turn spotty E2E testing into systematic behavior coverage: analyze the application, build a behavior-coverage matrix, detect gaps, duplicates, and false greens, then implement prioritized Playwright tests until meaningful gaps are exhausted.

## Core principles

These govern every phase. When a situation isn't covered by the instructions, decide with these.

1. **Behavior coverage is the only measure of done.** Not test counts, not route counts. A behavior is covered only when a test's *assertions* prove the user-visible outcome.
2. **Evidence or it isn't covered.** Every "covered" judgment cites the exact `file:line` of the assertions that prove it. Test titles are not evidence.
3. **Never modify application source code.** You may add or edit test code, test configuration, and files under `docs/e2e-coverage/`, and you may bootstrap Playwright if absent. That is all. If a new test reveals an application bug, record it in `findings.md`, mark the test `test.fixme` with a reason referencing the entry, and continue. Silently "fixing" the app to make a test pass is the one unrecoverable mistake here — it destroys the trust contract this skill runs on.
4. **State lives in artifacts, not memory.** All analysis state is written to `docs/e2e-coverage/` in the target project. An interrupted session must be able to resume from the artifacts. Never hold the matrix only in conversation.
5. **Be honest about incompleteness.** If gaps remain open when the loop ends, the report says so plainly. A false "all covered" claim is worse than an incomplete pass.

## Artifacts (working state)

All artifacts live in the target project under `docs/e2e-coverage/`:

- `matrix.md` — inventory, behavior list, the behavior × dimension matrix (before/after), legend
- `gaps.md` — prioritized gap list with resolution status (`implemented` / `known-failure` / `waived` / `open`)
- `findings.md` — application bugs discovered (known-failures), false greens detected, quarantined flaky tests

## Workflow

Six phases, each with entry/exit conditions. Read the reference file at the start of each phase.

| Phase | What happens | Reference |
|---|---|---|
| 0. Inventory | Stack, launch command, Playwright presence, routes, API, auth model, data/seed, existing tests + baseline run | `references/inventory.md` |
| 1. Behavior enumeration | List user behaviors (actor + action), tag dimensions, note risk | `references/behavior-matrix.md` |
| 2. Gap analysis | Map existing tests to cells with evidence; detect duplicates and false greens; prioritize gaps | `references/gap-analysis.md` |
| 3. Implementation | Write tests in priority order in small batches; triage failures; record bugs | `references/implementation.md` |
| 4. Re-analysis loop | Re-derive behaviors, re-map, rescan; up to 3 passes | `references/loop-termination.md` |
| 5. Final report | Before/after summary, findings, honest completion statement | `references/loop-termination.md` |

Phase gates in short:

- Phase 0 → 1: launch method known (or analysis-only mode declared), inventory written
- Phase 1 → 2: every behavior has dimension tags and a risk note
- Phase 2 → 3: every matrix cell has a status; `gaps.md` holds a prioritized list
- Phase 3 → 4: every gap is `implemented`, `known-failure`, or `waived`; new tests pass (fixme aside)
- Phase 4 → 5: termination criteria met (below)

## Completion criteria

Report complete only when BOTH hold:

1. Every matrix cell (except `n/a`) is in a terminal state: `implemented`, `known-failure`, or `waived` (with reason).
2. The most recent re-analysis pass produced zero new P0/P1 gaps.

Hard cap: 3 re-analysis passes. If the cap is hit, remaining items stay `open` in `gaps.md` and the final report explicitly says the pass was incomplete. Details in `references/loop-termination.md`.

## Priorities (used in Phases 2 and 3)

- **P0** — auth bypass, privilege escalation / cross-user access, data loss or corruption, security-relevant behavior, payments
- **P1** — core happy paths, user-impacting input validation, state transitions, persistence of critical entities
- **P2** — secondary flows, display-level error handling, edge-case UX

## First actions

1. Confirm the target project root and that it is a web app with a browser frontend. If it isn't, explain what this skill needs and stop.
2. Read `references/inventory.md` and begin Phase 0.
