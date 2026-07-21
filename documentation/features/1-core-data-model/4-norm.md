# 4. Norm

## What it does

Defines `Norm`, the composite dataclass representing a single Hohfeldian-typed deontic statement — the
central unit every other feature (the engine, correlative/delegation rules, conflict detection,
conflict resolution, chain validation, the resolution pipeline) operates on.

## Inputs / outputs

```python
Norm(
    id="n1",
    relation=Relation.PRIVILEGE,
    status=DeonticStatus.PERMITTED,
    subject="DataAgent",
    action="read",
    resource="/data/analytics/**",
    scope=Scope(contexts=frozenset({"production", "staging"})),
    provenance=Provenance(granted_by="SystemAdmin", granted_under="DataGovernancePolicy.section_3_2"),
    delegable=True,
)
```

## Edge cases and failure modes

- `counterparty` is required *in practice* for `RIGHT`/`POWER`/`IMMUNITY` relations (there's no
  directed duty/liability/disability without a named counterparty), but this story does **not** enforce
  that as a constructor-level invariant — cross-field, relation-specific validation belongs to the rules
  that consume `Norm`s in `3-hohfeldian-and-delegation-semantics`, not to the plain data type. Noted here
  so it isn't mistaken for an oversight.
- Two `Norm`s with identical field values (including nested `Scope`/`Condition`/`Provenance`) must be
  equal and share a hash — required for the engine's `dict[str, Norm]` norm store and any set-based
  dedup of derived norm-pointer facts.
- `given=None` means an unconditional (plain) obligation; `given=Condition(...)` means dyadic — this
  story only needs the field to exist and hold either value correctly, not interpret it.
- `liability_chain=()` (the default) means no delegation history yet.
- Construction with keyword arguments in any order must produce an equal `Norm` to the same arguments in
  a different order (dataclass equality is by field value, not by construction order — this is really a
  regression guard, not new behavior to build).

## Acceptance criteria

1. `Norm` is `@dataclass(frozen=True, slots=True)` with fields: `id: str`, `relation: Relation`,
   `status: DeonticStatus`, `subject: str`, `action: str`, `resource: str`,
   `counterparty: str | None = None`, `given: Condition | None = None`, `scope: Scope = Scope()`,
   `provenance: Provenance | None = None`, `delegable: bool = False`, `priority: float = 0.0`,
   `liability_chain: tuple[str, ...] = ()`.
2. A `Norm` instance is hashable.
3. Two `Norm`s built from identical field values (including equal nested `Scope`/`Provenance`/`Condition`
   instances) are `==` and share a hash.
4. A `Norm` with `given=None` and a `Norm` with `given=Condition(...)` are distinguishable via equality
   (they are not equal to each other even if every other field matches).

## Tasks

1. Add `Norm` to `src/deontic_reasoner/models.py`.
2. Write `tests/test_models_norm.py` covering acceptance criteria 1–4, including a case with all
   defaults and a case with every optional field populated.

## Deliverables

- `project/src/deontic_reasoner/models.py` (modified)
- `project/tests/test_models_norm.py` (new)

## Dependencies

- `1-core-enums-and-identifiers` (needs `Relation`, `DeonticStatus`)
- `3-norm-support-value-types` (needs `Condition`, `Scope`, `Provenance`)
