# 1. DONE - Norm Atom and Correlative Rule Grounding

## What it does

`atom_for_norm(norm) -> Atom` — a deterministic atom name for "this norm holds" — and
`ground_norm(norm, weight=CORRELATIVE_WEIGHT) -> tuple[Atom, Rule | None]`: for `RIGHT`/`POWER`/
`IMMUNITY` norms with a `counterparty`, also produces a Rule deriving the correlative duty/liability/
disability atom whenever the norm's own atom holds. Correlativity is represented as an ordinary
**weighted Rule**, not a `HardConstraint` — Hohfeld's correlativity is structurally strong, but modeling
it as an unconditionally-impossible-to-violate hard constraint would make it impossible to ever
*demonstrate* a duty being violated (exactly what `2-directed-obligation-regression`'s scenario needs to
be able to represent), so it uses the same weighted mechanism as every other rule, just with a
deliberately high default weight.

## Inputs / outputs

```python
norm = Norm(id="n1", relation=Relation.RIGHT, subject="FileAgent", action="write",
            resource="/out/report.csv", counterparty="filesystem")
atom, rule = ground_norm(norm)
# atom = atom_for_norm(norm)                       -- deterministic, e.g. "norm:n1"
# rule.body == frozenset({atom})
# rule.head == frozenset({<duty atom parameterized by "filesystem", "write", "/out/report.csv">})
# rule.weight == CORRELATIVE_WEIGHT
```

## Edge cases and failure modes

- `counterparty=None` on a `RIGHT`/`POWER`/`IMMUNITY` norm — no correlative rule is produced;
  `ground_norm` returns `(atom_for_norm(norm), None)`. A directed relation with nothing to direct it at
  derives nothing, silently, consistent with this reasoner's convention elsewhere (never raise on a
  non-applicable case, just don't fire).
- `PRIVILEGE` norms always return `(atom_for_norm(norm), None)` regardless of `counterparty` — privilege's
  own correlative (`no_right`) isn't exercised by any of this iteration's worked scenarios, so it isn't
  built (YAGNI; add it if a real need shows up).
- Two norms differing only in `subject`/`counterparty` must produce **different** correlative atoms —
  the atom naming is parameterized by the actual counterparty/action/resource, never accidentally
  shared between unrelated norms.
- `atom_for_norm` must be deterministic: the same `norm.id` always produces the same atom, across
  repeated calls and separate `ground_norm` invocations on an equal `Norm`.
- `CORRELATIVE_WEIGHT` is a module-level constant (high enough to dominate typical domain-rule weights
  by convention) — callers may override it via `ground_norm`'s `weight` parameter, consistent with
  `Requirements.md` → Questions: Trust and Governance #2 (whoever grounds/authors norms controls this
  weight, not the norm's own subject).

## Acceptance criteria

1. `atom_for_norm(norm)` is a deterministic function of `norm.id`.
2. `ground_norm(norm)` for `relation=RIGHT` with a `counterparty` returns
   `(atom_for_norm(norm), rule)` where `rule.body == frozenset({atom_for_norm(norm)})` and `rule.head`
   is a duty atom parameterized by `counterparty`/`action`/`resource`.
3. Same shape for `relation=POWER` (a liability atom) and `relation=IMMUNITY` (a disability atom).
4. `ground_norm(norm)` for `relation=PRIVILEGE` returns `(atom_for_norm(norm), None)`.
5. `ground_norm(norm)` for `RIGHT`/`POWER`/`IMMUNITY` with `counterparty=None` returns
   `(atom_for_norm(norm), None)`.
6. Two norms differing only in `subject`/`counterparty` produce different correlative atoms.
7. The returned rule's `weight` is `CORRELATIVE_WEIGHT` unless the caller passes an explicit override.

## Tasks

1. Create `src/deontic_reasoner/grounding.py`.
2. Define `CORRELATIVE_WEIGHT`, `atom_for_norm`, `ground_norm`.
3. Write `tests/test_grounding_norm.py` covering acceptance criteria 1–7.

## Deliverables

- `project/src/deontic_reasoner/grounding.py` (new)
- `project/tests/test_grounding_norm.py` (new)

## Dependencies

- `1-core-data-model` (feature) — needs `Norm`, `Relation`, `Atom`, `Rule`.
