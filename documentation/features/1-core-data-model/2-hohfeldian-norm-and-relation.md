# 2. Hohfeldian Norm and Relation

## What it does

Defines `Relation` (the four Hohfeldian incidents) and `Norm` — the dataclass grounding a single
Hohfeldian incident, matching `Requirements.md` FR10's exact signature
(`subject, relation, action, resource, condition`) plus the identifier, counterparty, and temporal
bounds every worked scenario needs.

## Inputs / outputs

```python
Norm(id="n1", relation=Relation.RIGHT, subject="FileAgent", action="write",
     resource="/out/report.csv", counterparty="filesystem")

Norm(id="n2", relation=Relation.PRIVILEGE, subject="DataAgent", action="read",
     resource="/data/analytics/**", valid_until=t0 + timedelta(hours=24))
```

## Edge cases and failure modes

- `counterparty=None` is legal at the data level for every relation, including `RIGHT`/`POWER`/
  `IMMUNITY`, which need one *in practice* to mean anything directed — validating that is deferred to
  `4-hohfeldian-grounding`, not enforced here (consistent with how relation-specific invariants were
  scoped out of the equivalent dataclass in the previous design).
- `valid_from`/`valid_until` both `None` means unbounded — this story only needs the fields to exist and
  hold `None` correctly; interpreting temporal bounds against a concrete request time is
  `5-scope-evaluation`'s job.
- `condition=None` means no extra applicability guard beyond the temporal bounds.
- `Norm` must be hashable (every field is `str`/`Relation`/`datetime`/`Atom`, or `None` — all hashable),
  and two `Norm`s built from identical field values are equal and share a hash.

## Acceptance criteria

1. `Relation` is a `StrEnum` with exactly `PRIVILEGE`, `RIGHT`, `POWER`, `IMMUNITY`.
2. `Norm` is `@dataclass(frozen=True, slots=True)` with `id: str`, `relation: Relation`,
   `subject: str`, `action: str`, `resource: str`, `counterparty: str | None = None`,
   `valid_from: datetime | None = None`, `valid_until: datetime | None = None`,
   `condition: Atom | None = None`.
3. `Norm` is hashable; two `Norm`s built from identical field values are equal and share a hash.
4. A `Norm` constructed with only the required fields (`id`, `relation`, `subject`, `action`,
   `resource`) leaves every optional field at its documented default (`None`).

## Tasks

1. Add `Relation` and `Norm` to `src/deontic_reasoner/models.py`.
2. Write `tests/test_models_norm.py` covering acceptance criteria 1–4.

## Deliverables

- `project/src/deontic_reasoner/models.py` (modified)
- `project/tests/test_models_norm.py` (new)

## Dependencies

- `1-propositional-core-types` — needs `Atom` for the `condition` field's type.
