# 2. Preference Ordering and Best-Worlds

## Name

`2-preference-ordering-and-best-worlds`

## Summary

The semantic engine's actual core: `violates(rule, world)`, `preferred(world_a, world_b)` under a
configurable criterion (subset/Pareto, count, or weighted count — defaulting to weighted count), and
`best_worlds(antecedent)` — the most-preferred worlds satisfying a given antecedent. This is what
directly replaces the previous (superseded) design's SAT-based conflict detection and XACML-style
combining algorithms: conflict resolution among competing norms falls out of which world violates the
least total weight, with no separate machinery needed. `best_worlds` scopes its enumeration to the atoms
actually mentioned in the loaded rule set/antecedent, not the full atom vocabulary the system has ever
seen, to stay tractable (Requirements.md → Questions: Data Representation #1).

## Requirements covered

- FR 5 — `violates(rule, world) -> bool`.
- FR 6 — `preferred(world_a, world_b) -> bool`, configurable criterion, weighted count as default.
- FR 7 — `best_worlds(antecedent) -> set[World]`, with scoped (not naive 2^n) enumeration.
- NFR 4 — determinism: identical inputs always produce identical `best_worlds` output.
- NFR 6 — tractability via atom-universe scoping.

## Dependencies

- `1-core-data-model` — `Atom`, `World`, `Rule`, `HardConstraint` must already exist.

## Stories

See `documentation/features/2-preference-ordering-and-best-worlds/` for the story breakdown (added by
`/stories`).
