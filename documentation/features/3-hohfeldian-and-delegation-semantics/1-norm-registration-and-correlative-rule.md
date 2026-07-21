# 1. Norm Registration and the Built-in Correlative Rule

## What it does

Two pieces that go together: `assert_norm(engine, norm)` — the public entry point for registering a
`Norm` into an engine's `norms` store and asserting its `Fact("norm", (norm.id,))` pointer into working
memory — and `build_correlative_rule(engine)`, a `Rule` bound to that specific engine whose consequent
inspects the referenced `Norm.relation` and derives the correct Hohfeldian correlative fact.

Pattern matching (feature 2) only sees flat `Fact` tuples of strings — it cannot branch on a `Norm`'s
`.relation` enum value directly. So this is necessarily **one** rule matching `Fact("norm", (Var("N"),))`,
whose consequent does plain Python dispatch on `engine.norms[bindings["N"]].relation` — not four separate
declaratively-matched rules. This mirrors the implementation spec's own note that `Fact("norm", (id,))`
is an opaque pointer into a parallel `dict[str, Norm]` store precisely so the matcher stays generic.

## Inputs / outputs

Correlative fact shapes (all `(party, action, resource, norm_id)`, chosen for consistency):

| Relation | Derived fact |
|---|---|
| `RIGHT` | `Fact("duty", (counterparty, action, resource, norm_id))` |
| `PRIVILEGE` | `Fact("no_right", (subject, action, resource, norm_id))` |
| `POWER` | `Fact("liability", (counterparty, action, resource, norm_id))` |
| `IMMUNITY` | `Fact("disability", (counterparty, action, resource, norm_id))` |

```python
assert_norm(engine, Norm(id="n1", relation=Relation.RIGHT, status=DeonticStatus.OBLIGATORY,
                          subject="FileAgent", action="write", resource="/out/report.csv",
                          counterparty="filesystem"))
engine.run()
# Fact("duty", ("filesystem", "write", "/out/report.csv", "n1")) in engine.facts
```

## Edge cases and failure modes

- `RIGHT`/`POWER`/`IMMUNITY` norms are directed relations — they need a `counterparty` to mean anything.
  A norm of one of these relations asserted with `counterparty=None` derives **no** correlative fact
  (silently — consistent with how pattern matching elsewhere in this reasoner never raises on a
  non-matching case, it just doesn't fire). Validating that such norms *should* carry a counterparty is
  out of scope (per `1-core-data-model/4-norm`'s note that relation-specific invariants aren't enforced
  at the dataclass level).
- `PRIVILEGE` never needs a `counterparty` — its correlative (`no_right`) is keyed on `subject` alone.
- Asserting the same `Norm` object twice (`assert_norm` called twice with an equal norm) and running must
  not produce a second, duplicate correlative fact — relies on feature 2's set-based dedup, verified here
  specifically for this rule.
- Two different norms of the same relation but different `id`s each derive their own correlative fact,
  distinguishable by the `norm_id` in the fourth position.

## Acceptance criteria

1. `assert_norm(engine, norm)` sets `engine.norms[norm.id] = norm` and adds `Fact("norm", (norm.id,))` to
   `engine.facts`.
2. `build_correlative_rule(engine)` returns a `Rule` whose single antecedent is
   `("norm", (Var("N"),))`.
3. Firing it for a registered `RIGHT` norm with a `counterparty` yields exactly
   `Fact("duty", (counterparty, action, resource, norm_id))`.
4. Firing it for a registered `PRIVILEGE` norm yields exactly
   `Fact("no_right", (subject, action, resource, norm_id))`.
5. Firing it for a registered `POWER` norm with a `counterparty` yields exactly
   `Fact("liability", (counterparty, action, resource, norm_id))`.
6. Firing it for a registered `IMMUNITY` norm with a `counterparty` yields exactly
   `Fact("disability", (counterparty, action, resource, norm_id))`.
7. A `RIGHT`/`POWER`/`IMMUNITY` norm registered with `counterparty=None` derives no correlative fact
   after `run()`.
8. Asserting the same norm twice and running still leaves exactly one correlative fact for it (no
   duplicate).

## Tasks

1. Create `src/deontic_reasoner/hohfeld.py`.
2. Implement `assert_norm(engine, norm)`.
3. Implement `build_correlative_rule(engine)` with the dispatch table above.
4. Write `tests/test_hohfeld_correlatives.py` covering acceptance criteria 3–8.

## Deliverables

- `project/src/deontic_reasoner/hohfeld.py` (new)
- `project/tests/test_hohfeld_correlatives.py` (new)

## Dependencies

- `2-forward-chaining-engine` (feature, all 3 stories) — needs `ReasonerEngine`, `Rule`, `Var`.
- `1-core-data-model` (feature) — needs `Norm`, `Relation`, `Fact`.
