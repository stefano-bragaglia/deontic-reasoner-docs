# 3. Hohfeldian and Delegation Semantics

## Name

`3-hohfeldian-and-delegation-semantics`

## Summary

The domain-specific rules that make this a *Hohfeldian deontic* reasoner rather than a generic
forward-chaining engine. Two rule groups, both built-in (always applied, no per-domain opt-in — per
Requirements.md → Questions: Built-in Semantics #3 and #4):

1. **Correlative derivation** — asserting a norm of a given `Relation` automatically derives its
   Hohfeldian correlative fact: Right → counterparty Duty, Privilege → No-Right, Power → Liability,
   Immunity → Disability.
2. **Delegation semantics** — a single `delegate(A, B, permission)` fact automatically derives A's audit
   duty, A's dyadic revoke-on-violation duty (`O(revoke | violation)`, not a material conditional — this
   is what avoids the classical contrary-to-duty failure modes, i.e. Chisholm's paradox), and A's
   liability; and delegation is automatically scope-narrowing (a re-delegated scope is intersected with
   the delegator's own, so a delegatee can never end up with a *wider* scope than its delegator held).

This is one feature, not two, because both rule groups are the same shape of thing — built-in,
always-on forward-chaining rules that make a norm's stated `Relation`/delegation fact *mean* something
beyond its bare data — and both depend on the engine (`2-forward-chaining-engine`) existing to fire them.

## Requirements covered

- FR 2 (behavior half) — automatic correlative derivation for all four Hohfeldian relations.
- FR 3 — dyadic/conditional obligation representation and correct firing behavior (a dyadic obligation's
  consequent only derives when its `given` condition is actually satisfied, never vacuously).
- FR 5 — delegation obligations (audit, dyadic revoke-on-violation, liability) derived automatically from
  a single `delegate(...)` fact, plus automatic scope-narrowing on re-delegation.
- Acceptance criteria: implementation spec worked scenarios §14.3 (delegation derives its
  contrary-to-duty obligations automatically), §14.7 (scope-narrowing on re-delegation), and §14.8
  (Chisholm's paradox regression — the dyadic condition must not degrade to material-conditional
  semantics).

## Dependencies

- `1-core-data-model` — `Norm.relation`, `Norm.given`, `Norm.delegable`, `Scope` must exist.
- `2-forward-chaining-engine` — these rules are `Rule` objects fired by `ReasonerEngine`; nothing to
  build this feature on without the engine.

## Stories

See `documentation/features/3-hohfeldian-and-delegation-semantics/` for the story breakdown (added by
`/stories`).
