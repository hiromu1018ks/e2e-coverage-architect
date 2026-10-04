# Phase 4 — Re-analysis loop & Phase 5 — Final report

## Why re-analyze

The first pass misses things: behaviors you didn't spot, dimensions marked `n/a` too eagerly, gaps that turned out untestable. The loop exists to catch your own blind spots — treat each pass as an attempt to prove your previous pass wrong.

## Procedure (one pass)

1. **Re-derive behaviors from the code, fresh.** Don't just re-read your old list — walk the routes and endpoints again as if seeing them for the first time. New rows go into the matrix.
2. **Re-map cells including all tests written so far**, applying the same evidence rule.
3. **Re-scan:** any uncovered/partial cells remaining? Any false greens among your own tests (rushed tests inherit the same diseases — apply the checklist to them too)?
4. Update the matrix (AFTER table) and `gaps.md`.

## Termination

Report complete when BOTH:

1. All cells (except `n/a`) are terminal: `implemented` / `known-failure` / `waived` (reason required)
2. This pass produced **zero new P0/P1 gaps**

Hard cap: **3 passes**. On hitting the cap, remaining gaps stay `open` in `gaps.md` and the outcome is "incomplete".

An incomplete outcome is acceptable and must be stated plainly; a false "complete" is not. The completion claim must be checkable from `matrix.md` and `gaps.md` alone — anyone reading them should be able to verify it without asking you.

## Phase 5 — Final report (in chat)

```markdown
# E2E coverage report — <project>

## Outcome
<complete | incomplete (N open gaps)> — against the termination criteria

## Coverage
- Behaviors: N | Cells resolved: X/Y (Z n/a)
- Tests added: N (P0: a, P1: b, P2: c) | fixme'd: N

## False greens found
<list with test references, or "none detected">

## Application bugs (known-failures — NOT fixed)
findings#N — <title> — <one-line repro>

## Waived cells
<behavior × dimension — reason>

## Open gaps (if incomplete)
<ID — priority — why it remains open>

## Artifacts
docs/e2e-coverage/{matrix,gaps,findings}.md
```

Rules for the report: application bugs are reported, never fixed. Be specific — "improved coverage" is noise; "added 9 tests; user-vs-admin authorization is now asserted for delete and the admin user list" is signal.
