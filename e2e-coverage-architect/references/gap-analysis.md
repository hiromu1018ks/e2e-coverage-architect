# Phase 2 — Gap analysis: mapping, false greens, priorities

## Map tests to cells

For each existing test file:

1. Read the test body. Ask: which behavior row(s) and dimension(s) do its **assertions** actually prove? Titles and test names are hints, not evidence — a test titled "admin can delete users" whose body only waits for a URL proves nothing about deletion.
2. Mark cells `covered(file:line)` where assertions genuinely prove the outcome.
3. If assertions prove only part of the cell's meaning, mark `partial(file:line, what's missing)`.

The evidence rule: a cell is covered only if you can name the cited assertion that would **fail if the behavior broke**. If you can't, it isn't covered.

## False-green checklist

Apply to every test that appears to cover a cell. Any hit → downgrade the cell to `partial`, and record the test in `findings.md` under "False greens" with the reason.

1. **No meaningful assertions** — only navigation/waits (`waitForURL`, `toHaveURL`, `waitForSelector`, bare `waitForTimeout`). Reaching a page is not an outcome.
2. **Absence-of-error only** — asserts no error text or status < 500. Proves "didn't crash", not "worked".
3. **Skipped or fixme** — `test.skip` / `test.fixme` / conditional skip. A skipped test covers nothing, whatever its title claims.
4. **Asserting seed data the test didn't create** — verifies the fixture, not the behavior. The test never drove the action.
5. **Overly broad matchers** — `toHaveCount(0)`, `toBeVisible()` on generic containers, where a specific outcome should be pinned. Broad passes hide regressions.
6. **Order / shared-state dependence** — depends on another test's data; breaks when run alone. The coverage exists but is fragile → partial.
7. **No persistence re-check** — action asserted only in the immediate UI response; no reload/refetch confirms durability (relevant to the persistence dimension only).

Your own new tests are not exempt — Phase 4 re-applies this checklist to everything, including what you wrote.

## Duplicates

Group tests whose assertions cover the same cell. Report the clusters in `findings.md` with a one-line "what to consider consolidating" note. Never delete or merge existing tests yourself — that is the user's call.

## Prioritization

Assign each uncovered/partial cell a priority:

- **P0** — auth bypass, privilege escalation, cross-user access, data loss/corruption, security, payments
- **P1** — core happy paths, user-impacting validation, state transitions, persistence of critical entities
- **P2** — secondary flows, display-level errors, edge-case UX

Tie-breakers within a priority: how often users hit the flow, and blast radius if it breaks.

## Output: gaps.md

```markdown
# Coverage gaps (prioritized)

| ID | Behavior | Dimension | Priority | Status | Notes |
|---|---|---|---|---|---|
| G1 | User deletes another user's post | authz | P0 | open | expect 403 + post remains in list |
| G2 | User edits own post title | happy | P1 | open | edit form → save → list shows new title |
```

Status: `open` → later `implemented(file:line)` / `known-failure(findings#N)` / `waived(reason)`.

Include a one-line suggested test approach per gap in Notes — Phase 3 will thank you.

## Exit checklist

- [ ] Every cell has a status, with evidence for covered/partial
- [ ] False-green hits recorded in `findings.md`
- [ ] `gaps.md` prioritized, deduplicated, with test-approach sketches
