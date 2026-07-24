# 6. DONE - Forward Chaining and Delegation

## Name

`6-forward-chaining-and-delegation`

## Summary

The integration feature: a fixed-point loop wrapped *around* the query engine (features 2–5) — the
13-predicate core answers "what follows from the current rule set," this feature answers "what new
facts get added, and keep going until nothing more follows." Covers **powers as norm-generating acts**
(exercising a power creates a new norm fact, via an ordinary domain-specific fact as a conditional
rule's body — no dedicated predicate needed, generalizing delegation's own mechanism) and
**delegation's derived oversight obligations** (`delegated(delegatee, delegator, norm)` triggers audit,
revoke-on-violation, and liability duties through the same conditional-rule mechanism as any other rule,
plus automatic scope-narrowing on re-delegation). This is necessarily the last feature — every one of
the preceding five has to exist before there's anything to wrap a fixed-point loop around.

## Requirements covered

- FR 12 — delegation's oversight obligations, derived automatically via the ordinary conditional-rule
  mechanism (no separate machinery), including automatic scope-narrowing on re-delegation.
- FR 14 — the forward-chaining fixed-point loop itself, covering both power-exercise-derived norms
  (14a) and delegation's derived obligations (14b); bounded (NFR 5) rather than looping forever on a
  malformed/oscillating rule set.
- Acceptance criteria (Requirements.md → Questions: Scenario Coverage #5's adaptation):
  - §14.3 — delegation derives its oversight obligations automatically from a single `delegated` fact,
    with no manual authoring of the derived norms.
  - §14.5 — a broken delegation link (a missing `valid_link` fact partway up the chain) denies a
    downstream permission, via ordinary conditional-rule matching — no dedicated chain-walking module.
  - §14.6 — split: the power-creates-a-norm half translates directly (this is literally requirement
    14a); the immunity-blocks-revocation half is reduced to asserting an `IMMUNITY` norm fact is
    representable and forward-chainable, not that it behaviorally blocks anything (immunity's real
    teeth are later-iteration work, per `Description.md` → Open questions and Requirements.md →
    Questions: Power and Immunity Semantics #7).
  - §14.7 — scope-narrowing on re-delegation: a delegatee's granted scope is automatically intersected
    with the delegator's own, never wider.

## Dependencies

- `1-core-data-model`, `2-preference-ordering-and-best-worlds`,
  `3-obligation-and-permissibility-queries`, `4-hohfeldian-grounding`, `5-scope-evaluation` — this
  feature has no content of its own beyond wrapping a fixed-point loop around all five.

## Stories

See `documentation/features/6-forward-chaining-and-delegation/` for the story breakdown (added by
`/stories`).
