# 7. Delegation Chain Validation

## Name

`7-delegation-chain-validation`

## Summary

Validates that a delegation/liability chain is actually sound before a request that depends on it is
granted: cycle detection over the delegation graph via stdlib `graphlib.TopologicalSorter` (a cycle is
either a modeling error or an authority-laundering attempt — treated as a hard validation failure), plus
a linear chain-validity walk (each agent has exactly one direct delegator, so validity is a single path
once the graph is known acyclic) that catches a *broken* link — a grantor that no longer actually holds
the power it purportedly delegated — regardless of whether the leaf permission itself looks well-formed.

## Requirements covered

- FR 13 — validate delegation/liability chains: detect cycles structurally, detect broken links, deny
  the dependent request regardless of the leaf permission's own validity.
- Acceptance criteria: implementation spec worked scenario §14.5 (broken delegation chain — a revoked
  grant partway up the chain must deny a request even though the requester's own norm is otherwise
  well-formed).

## Dependencies

- `1-core-data-model` — delegation edges are `(delegatee, delegator)` tuples derived from `Fact`/`Norm`
  data.
- `3-hohfeldian-and-delegation-semantics` — this feature validates the delegation chains that feature 3's
  delegation rule produces; without delegation facts existing, there's no chain to validate.

## Stories

See `documentation/features/7-delegation-chain-validation/` for the story breakdown (added by
`/stories`).
