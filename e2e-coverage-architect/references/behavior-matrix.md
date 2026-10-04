# Phase 1 — Behavior enumeration & the matrix

## What is a "behavior"

A behavior is a user-action unit: **actor + action (+ expected outcome)**. "Regular user deletes own post", "Admin deletes any user's post", "Visitor opens a protected page".

The same action with a different actor is a different row — authorization differences live in the rows, everything else in the dimension columns. "Admin deletes a post" and "Regular user deletes a post" fail differently and must be tested differently.

Derive behaviors by walking:

1. Each page/route from the inventory: what actions are reachable there (buttons, forms, links that change state)?
2. Each mutating API endpoint: which UI flow triggers it?
3. Auth events themselves: login, logout, session expiry if observable.

Do not invent features that don't exist. Rows come from code, not imagination.

## The six dimensions

| Dimension | Meaning | Example (post deletion) |
|---|---|---|
| happy path | the action succeeds as intended | delete own post → gone from list |
| validation | invalid input is rejected with feedback | (n/a for delete; for create: empty title → error shown) |
| authorization | wrong-role / other-user attempts are blocked | user B deletes user A's post → 403, post remains |
| error handling | server/edge failures surface usefully | API failure → user sees an error, not a silent hang |
| state transition | surrounding state changes correctly | logged-out visitor redirected to login; returns after login |
| persistence | the change survives reload / re-login | deleted post still gone after reload |

Not every dimension applies to every behavior. Mark inapplicable cells `n/a` with a short reason — but be honest with yourself: "n/a because it's awkward to test" is not a reason.

## Risk notes

One short note per behavior: if this breaks, is it data loss, security breach, money, or mere annoyance? This feeds prioritization in Phase 2. A wrong-role delete failing open is P0 regardless of how obscure the page is.

## Matrix format (in matrix.md)

```markdown
## Behavior × Dimension matrix — BEFORE

| # | Behavior | happy | validation | authz | error | state | persist |
|---|---|---|---|---|---|---|---|
| 1 | Visitor signs in with valid credentials | ✅ auth.spec.js:12 | n/a | n/a | — | — | — |
| 2 | User creates a post | ✅ posts.spec.js:10 | — | n/a | — | — | — |
```

Status codes:

- `✅ covered(file:line)` — assertions genuinely prove the outcome
- `◐ partial(file:line, what's missing)`
- `— uncovered`
- `n/a(reason)`

Keep an "AFTER" copy of the table updated as Phase 3 resolves cells. Terminal states used in the AFTER table: `implemented(file:line)`, `known-failure(findings#N)`, `waived(reason)`.

## Exit checklist

- [ ] Every behavior has a row with actor + action
- [ ] Risk note per behavior
- [ ] Deliberate about near-duplicate rows: admin/user variants stay separate on purpose; true duplicates merged consciously
- [ ] Cells ready for Phase 2 evaluation
