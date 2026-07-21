# 1. Core Enums and Identifiers

## What it does

Defines `Relation` and `DeonticStatus` as `enum.StrEnum` types, and `Agent`/`Resource` as minimal,
frozen, hashable identifier dataclasses.

## Inputs / outputs

- `Relation.PRIVILEGE`, `Relation.RIGHT`, `Relation.POWER`, `Relation.IMMUNITY` — `.value` is the
  lowercase name (`"privilege"`, `"right"`, `"power"`, `"immunity"`).
- `DeonticStatus.OBLIGATORY`, `DeonticStatus.PERMITTED`, `DeonticStatus.FORBIDDEN`.
- `Agent(id="DataAgent")`, `Resource(id="/data/analytics/report.csv")`.

## Edge cases and failure modes

- Two `Agent` instances constructed with the same `id` must be equal and share a hash (value equality,
  not identity) — needed so agents can be de-duplicated and used as dict/set keys.
- `Agent("A") != Resource("A")` — same id string, different types, must **not** compare equal.
- Empty-string `id` is a legal value at the dataclass level (no validation here — nothing in
  `Requirements.md` calls for id format validation).
- Mutating `.id` after construction must raise `dataclasses.FrozenInstanceError`.

## Deliberate simplification vs. the implementation spec

The implementation spec's data model (§5) gives `Agent`/`Resource` an `attributes: Mapping[str, object]`
field for arbitrary per-entity metadata. This is dropped here: nothing in `Requirements.md`'s functional
requirements or the nine worked test scenarios needs per-agent attributes, and a mutable-mapping field
would break the frozen dataclass's hashability (a plain `dict` isn't hashable, so `hash(Agent(...))`
would raise `TypeError` the moment `attributes` is non-empty) — hashability is required since instances
must live in `set`s/be `dict` keys elsewhere in the reasoner. If a real need for per-agent metadata shows
up in a later story or iteration, add it then with a hashable representation (e.g. a `frozenset` of
`(key, value)` pairs), not now.

## Acceptance criteria

1. `Relation` is a `StrEnum` with exactly the members `PRIVILEGE`, `RIGHT`, `POWER`, `IMMUNITY`.
2. `DeonticStatus` is a `StrEnum` with exactly the members `OBLIGATORY`, `PERMITTED`, `FORBIDDEN`.
3. `Agent` and `Resource` are each `@dataclass(frozen=True, slots=True)` with a single field `id: str`.
4. `Agent("A") == Agent("A")` is `True`; `hash(Agent("A")) == hash(Agent("A"))` is `True`.
5. `Agent("A") != Resource("A")`.
6. Assigning to `.id` on an existing `Agent`/`Resource` instance raises `dataclasses.FrozenInstanceError`.

## Tasks

1. Create `src/deontic_reasoner/models.py`.
2. Define `Relation(StrEnum)` and `DeonticStatus(StrEnum)`.
3. Define `Agent` and `Resource` as frozen, slotted dataclasses with a single `id: str` field.
4. Write `tests/test_models_identifiers.py` covering acceptance criteria 1–6.

## Deliverables

- `project/src/deontic_reasoner/models.py` (new)
- `project/tests/test_models_identifiers.py` (new)

## Dependencies

None — first story in this feature.
