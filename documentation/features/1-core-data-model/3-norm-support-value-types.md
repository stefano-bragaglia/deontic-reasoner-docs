# 3. Norm-Support Value Types — Condition, Scope, Provenance

## What it does

Defines the three supporting value types a `Norm` embeds: `Condition` (a named predicate guard, used
both for `Scope.conditions` and for a dyadic obligation's `given`), `Scope` (temporal/context/condition
bounds), and `Provenance` (who granted a norm, under what authority).

## Inputs / outputs

- `Condition("violation", ("DataAgent", "R1"))`
- `Scope(valid_from=dt1, valid_until=dt2, contexts=frozenset({"production", "staging"}), conditions=(cond,))`
- `Provenance(granted_by="SystemAdmin", granted_under="DataGovernancePolicy.section_3_2")`

## Edge cases and failure modes

- A default-constructed `Scope()` (no bounds, no contexts, no conditions) means "unbounded" —
  everything about it is a permissive default; this story only defines the data shape, not the
  interpretation logic (interpreting an unbounded `Scope` as "always in scope" is
  `4-condition-and-scope-evaluation`'s job, not this one's).
- `Scope.contexts` must be a `frozenset[str]`, not a `set[str]` — a plain `set` field would make `Scope`
  (and therefore any `Norm` embedding it) unhashable.
- Cross-field sanity (e.g. `valid_from` after `valid_until`) is **not** validated by this story — nothing
  in `Requirements.md` calls for scope-sanity validation, and no downstream feature currently needs it;
  noted here so it isn't silently assumed to exist.
- `Condition` with zero args (`Condition("business_hours")`) must construct correctly.

## Acceptance criteria

1. `Condition` is `@dataclass(frozen=True, slots=True)` with `predicate: str` and `args: tuple[str, ...] = ()`.
2. `Scope` is `@dataclass(frozen=True, slots=True)` with `valid_from: datetime | None = None`,
   `valid_until: datetime | None = None`, `contexts: frozenset[str] = frozenset()`,
   `conditions: tuple[Condition, ...] = ()`.
3. `Provenance` is `@dataclass(frozen=True, slots=True)` with `granted_by: str` and `granted_under: str`.
4. `Scope() == Scope()` and both share a hash (default-constructed instances are equal).
5. `Scope` instances are hashable when `contexts` is non-empty (proving `frozenset`, not `set`, was
   actually used).
6. `Condition` instances are hashable and support empty `args`.

## Tasks

1. Add `Condition`, `Scope`, `Provenance` to `src/deontic_reasoner/models.py`.
2. Write `tests/test_models_scope.py` covering acceptance criteria 1–6.

## Deliverables

- `project/src/deontic_reasoner/models.py` (modified)
- `project/tests/test_models_scope.py` (new)

## Dependencies

None — `Condition`, `Scope`, `Provenance` are self-contained within this story (`Scope` embeds
`Condition`, both defined here).
