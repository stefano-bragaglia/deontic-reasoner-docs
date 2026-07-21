# 5. JSON Round-Trip

## What it does

Adds serialization so every core dataclass (`Agent`, `Resource`, `Fact`, `Condition`, `Scope`,
`Provenance`, `Norm`) converts to and from plain JSON-compatible data, with `datetime` fields as
ISO-8601 strings and enum fields as their `.value` — no custom binary format, per `Requirements.md`
NFR 7.

## Inputs / outputs

- `to_dict(norm) -> dict` — a JSON-serializable dict (nested dataclasses/enums/datetimes/tuples/
  frozensets all converted to plain `dict`/`list`/`str`/`int`/`float`/`bool`/`None`).
- `from_dict(Norm, data) -> Norm` (or an equivalent per-type reconstruction function — exact dispatch
  shape is a Stage B implementation detail) reconstructing an object equal to the original.
- Round-trip example: `from_dict(Norm, to_dict(norm)) == norm`.

## Edge cases and failure modes

- `None`-valued optional fields (`counterparty`, `given`, `provenance`) must round-trip as JSON `null` /
  Python `None` — not dropped from the dict, not coerced into a sentinel string.
- `Scope.contexts` (a `frozenset[str]`) must serialize as a JSON array and deserialize back into a
  `frozenset`, not a `list` — otherwise the reconstructed `Scope` would fail equality/hash against the
  original.
- Nested types must recurse correctly: `Norm.scope.conditions` is a tuple of `Condition`, each of which
  must itself round-trip.
- The all-defaults instance of every type (e.g. `Scope()`, a `Norm` with every optional field at its
  default) must round-trip correctly, not just a "fully populated" instance — defaults are exactly where
  `None`-handling bugs hide.
- `to_dict(x)` output must be accepted by `json.dumps()` with **no custom encoder** — i.e. it must
  contain only `dict`, `list`, `str`, `int`, `float`, `bool`, `None` after conversion; no raw `datetime`,
  `enum` member, `tuple`, or `frozenset` may remain in the output.

## Acceptance criteria

1. For each of the six dataclasses (`Agent`, `Resource`, `Fact`, `Condition`, `Scope`, `Provenance`) and
   for `Norm`, `from_dict(to_dict(x)) == x` holds for both an all-defaults instance and a fully-populated
   instance.
2. `json.loads(json.dumps(to_dict(x)))` succeeds with no custom `json.JSONEncoder` for every type above.
3. A `datetime` field serializes via `.isoformat()` and deserializes via `datetime.fromisoformat()`,
   verified on `Scope.valid_from`/`valid_until`.
4. `Relation` and `DeonticStatus` fields serialize as their `.value` string and deserialize back to the
   correct enum member, verified on `Norm.relation`/`Norm.status`.
5. `Scope.contexts` round-trips through a JSON array back into a `frozenset[str]` equal to the original.

## Tasks

1. Create `src/deontic_reasoner/serialization.py` with `to_dict`/`from_dict` functions (or per-type
   `*_to_dict`/`*_from_dict` functions — whichever keeps the dispatch simplest; decide in Stage B).
2. Handle the three non-trivial conversions explicitly: `datetime` ↔ ISO string, enum member ↔ `.value`,
   `frozenset`/`tuple` ↔ JSON array.
3. Write `tests/test_serialization.py` covering acceptance criteria 1–5 for all seven types, each with
   both an all-defaults and a fully-populated instance.

## Deliverables

- `project/src/deontic_reasoner/serialization.py` (new)
- `project/tests/test_serialization.py` (new)

## Dependencies

- `1-core-enums-and-identifiers`, `2-working-memory-fact`, `3-norm-support-value-types`, `4-norm` — every
  type must exist before it can be serialized.
