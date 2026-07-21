# 1. DONE - Core Data Model

## Name

`1-core-data-model`

## Summary

The foundational data every other feature operates on: `Atom`, `World` (`frozenset[str]` of true
atoms), `Rule` (`body`, `head`, `weight`), `HardConstraint`, and the Hohfeldian `Norm`/`Relation` types
(`privilege`/`right`/`power`/`immunity`). This exists first because nothing else — preference ordering,
queries, grounding, scope evaluation, forward chaining — has anything to operate on until these types
exist, exactly the same reason the equivalent feature existed first in the previous (superseded) design.
Also covers JSON round-tripping, since norms/rules must be authorable and auditable as plain JSON with no
custom binary format.

## Requirements covered

- FR 1 — atoms as the base propositional vocabulary.
- FR 2 — `World = frozenset[str]`, hashable and immutable by construction (Requirements.md → Questions:
  Data Representation #1).
- FR 3 — conditional rules as `(body, head, weight)` triples; `weight` is data this feature merely
  carries — the trust-boundary requirement on who may set it (Questions: Trust and Governance #2)
  applies to whatever stores/authors rules, not to this feature's plain dataclass.
- FR 4 — hard constraints, kept in the data model even though unexercised by this iteration's worked
  scenarios (Questions: Scenario Coverage #4).
- FR 10 (representation half), FR 11 (representation half) — `Norm`/`Relation` as first-class,
  JSON-serializable data; the *behavior* of grounding a norm as an atom is `4-hohfeldian-grounding`'s
  job, this feature only needs the type to exist and round-trip.
- FR 15 — every core dataclass is JSON-serializable (`dataclasses.asdict`/a small `from_dict`), no
  custom binary format.

## Dependencies

None — this is the first feature; everything else depends on it.

## Stories

See `documentation/features/1-core-data-model/` for the story breakdown (added by `/stories`).
