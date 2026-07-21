# 3. JSON Round-Trip

## What it does

Adds serialization so `Rule`, `HardConstraint`, and `Norm` (plus `World` directly, since it's central
enough to the public API to need its own conversion) convert to and from plain JSON-compatible data —
`datetime` fields as ISO-8601 strings, enum fields as their `.value`, `frozenset` fields as JSON arrays —
with no custom binary format, per `Requirements.md` FR 15.

## Inputs / outputs

- `to_dict(x) -> dict` / `from_dict(Type, data) -> Type` for `Rule`, `HardConstraint`, `Norm`.
- `world_to_list(world: World) -> list[str]` / `world_from_list(data: list[str]) -> World` for `World`
  specifically, since it's a bare type alias rather than a dataclass `to_dict` can introspect.

## Edge cases and failure modes

- `Rule.body`/`Rule.head` and `HardConstraint.forbidden` (all `frozenset[Atom]`) must round-trip
  through a JSON array back to a `frozenset` equal to the original — order in the JSON array is
  irrelevant to the resulting set's equality, but the round-trip must still land on the exact same
  `frozenset`, not a `list`.
- `Norm`'s `None`-valued optional fields (`counterparty`, `valid_from`, `valid_until`, `condition`) must
  round-trip as JSON `null`/Python `None`, not dropped from the dict or coerced into a sentinel string.
- `Norm.valid_from`/`valid_until` serialize via `.isoformat()` and deserialize via
  `datetime.fromisoformat()`.
- `Norm.relation` serializes as its `.value` string and deserializes back to the correct `Relation`
  member.
- The all-defaults instance of each type (e.g. a `Norm` with every optional field at its default) must
  round-trip correctly, not just a fully-populated instance — defaults are exactly where `None`-handling
  bugs hide.
- `to_dict(x)` output must be accepted by `json.dumps()` with **no custom encoder** for all three types.
- `world_to_list`/`world_from_list` round-trips a `World` through a JSON array back to an equal
  `frozenset`, the same guarantee `Rule.body`/`head` get via the generic dataclass path.

## Acceptance criteria

1. For `Rule`, `HardConstraint`, and `Norm`: `from_dict(Type, to_dict(x)) == x` holds for both an
   all-defaults instance and a fully-populated instance.
2. `json.loads(json.dumps(to_dict(x)))` succeeds with no custom `json.JSONEncoder`, for all three types.
3. `Rule.body`/`head` and `HardConstraint.forbidden` round-trip through a JSON array back to an equal
   `frozenset`.
4. `Norm`'s `None`-valued optional fields round-trip as `None`, not dropped.
5. `Norm.valid_from`/`valid_until` round-trip via `.isoformat()`/`datetime.fromisoformat()`.
6. `Norm.relation` round-trips via its `.value` string.
7. `world_from_list(world_to_list(w)) == w` for both an empty `World` and a non-empty one.

## Tasks

1. Create `src/deontic_reasoner/serialization.py` with `to_dict`/`from_dict` (or per-type functions —
   whichever keeps the dispatch simplest, decide in Stage B) and `world_to_list`/`world_from_list`.
2. Write `tests/test_serialization.py` covering acceptance criteria 1–7 for all three dataclass types
   plus `World`, each with both an all-defaults and a fully-populated instance where applicable.

## Deliverables

- `project/src/deontic_reasoner/serialization.py` (new)
- `project/tests/test_serialization.py` (new)

## Dependencies

- `1-propositional-core-types`, `2-hohfeldian-norm-and-relation` — every type must exist before it can
  be serialized.
