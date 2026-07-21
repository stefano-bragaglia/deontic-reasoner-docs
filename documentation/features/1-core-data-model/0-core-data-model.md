# 1. Core Data Model

## Name

`1-core-data-model`

## Summary

The foundational data types every other feature builds on: `Agent`, `Resource`, `Fact`, `Norm`, `Scope`,
`Condition`, `Provenance`, and the `Relation`/`DeonticStatus` enums, all as `frozen=True, slots=True`
dataclasses (immutable, hashable — required since facts and norms live in `set`s). This exists first
because nothing else — the rule engine, conflict detection, chain validation, the resolution pipeline —
has anything to operate on until these types exist. Also covers JSON round-tripping
(`dataclasses.asdict`/a small `from_dict` per type), since norms/facts must be authorable and auditable
as plain JSON with no custom binary format.

## Requirements covered

- FR 1 — first-class, JSON-serializable data for agents, resources, facts, norms.
- FR 2 (representation half) — `Relation` enum (privilege/right/power/immunity) and `DeonticStatus` enum
  (obligatory/permitted/forbidden) as fields on `Norm`.
- FR 3 (representation half) — `Norm.given: Condition | None` as the slot for dyadic/conditional
  obligations (the derivation logic that makes dyadic conditions behave correctly is `3-hohfeldian-and-delegation-semantics`;
  this feature only needs the field to exist and round-trip).
- NFR 1 — standard-library-only (`dataclasses`, `enum`, `datetime`, `typing`).
- NFR 6 — JSON round-trip for every core dataclass.
- NFR 8 (trust boundary) — the shape that makes `Norm.priority`/`Norm.provenance` inspectable, which is
  what lets a future norm-store enforce the "only the granting authority sets priority" rule; enforcing
  it is explicitly out of this reasoner's scope (Requirements.md NFR 8), this feature just carries the
  fields.

## Dependencies

None — this is the first feature; everything else depends on it.

## Stories

See `documentation/features/1-core-data-model/` for the story breakdown (added by `/stories`).
