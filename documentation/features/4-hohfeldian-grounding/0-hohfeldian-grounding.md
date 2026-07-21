# 4. Hohfeldian Grounding

## Name

`4-hohfeldian-grounding`

## Summary

Bridges the propositional core to multi-agent, *directed* obligations — the gap plain `O`/dyadic-`O(q|p)`
can't express on its own. `norm(subject, relation, action, resource, condition)` grounds a Hohfeldian
incident as an atom the core can reason over; `counterparty(norm, agent)` names who owes the duty, is
liable, or is disabled for a `right`/`power`/`immunity` norm. `relation=right` is what actually produces
a *directed* obligation (owed by a specific counterparty to a specific holder) once fed through
`3-obligation-and-permissibility-queries`.

## Requirements covered

- FR 10 — grounding a Hohfeldian incident as an atom-bearing fact.
- FR 11 — the correlative bearer via `counterparty(norm, agent)`.
- Acceptance criteria: §14.2 — a `right` norm with a named counterparty produces a directed obligation
  (the counterparty's duty), verified via the obligation query — demonstrating the reasoner can express
  *directed* obligations that plain `O`/dyadic-`O(q|p)` cannot.

## Dependencies

- `1-core-data-model` — needs `Norm`, `Relation`, `Atom`.
- `3-obligation-and-permissibility-queries` — grounding is only verifiable by actually querying whether
  the correlative obligation holds; nothing to test this feature against without the query layer.

## Stories

See `documentation/features/4-hohfeldian-grounding/` for the story breakdown (added by `/stories`).
