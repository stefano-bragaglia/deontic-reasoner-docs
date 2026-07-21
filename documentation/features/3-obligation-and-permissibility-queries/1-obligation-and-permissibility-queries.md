# 1. Obligation and Permissibility Queries

## What it does

`is_obligatory(b, given, rules, hard_constraints=(), criterion=WEIGHTED_COUNT) -> bool` — does `b` hold
at **every** world in `best_worlds(given, ...)`? — and `is_permitted(b, given, rules, ...) -> bool` —
does `b` hold at **some** world in `best_worlds(given, ...)`? These are Hansson's dyadic `O(q|p)` and its
permission dual, made concrete and callable.

## Inputs / outputs

```python
is_obligatory(b=frozenset({"audit"}), given=frozenset({"delegated"}), rules=[...])       # -> bool
is_permitted(b=frozenset({"send_external"}), given=frozenset({"request_received"}), rules=[...])  # -> bool
```

Both `b` and `given` are `frozenset[Atom]` (a conjunction), matching `Rule.body`/`head` and
`best_worlds`'s own `antecedent` parameter type — "does `b` hold at world `w`" means `b <= w`.

## Edge cases and failure modes

- **Vacuous case**: when `best_worlds(given, ...)` is empty (e.g. `given` is inherently impossible under
  a hard constraint), `is_obligatory` returns **`True`** (universal quantification over an empty set is
  vacuously true) and `is_permitted` returns **`False`** (existential quantification over an empty set is
  vacuously false). This asymmetry is deliberate, classical-logic-consistent behavior — not a bug — and
  is documented explicitly here so it's never "fixed" into something inconsistent later.
- `b = frozenset()` (the empty conjunction) is vacuously satisfied by every world (same convention
  `Rule.head`'s emptiness already uses) — so `is_obligatory(frozenset(), given, rules)` is `True`
  whenever `best_worlds(given, ...)` is non-empty (it holds trivially at every world in it), regardless
  of `given`/`rules` content.
- Omitting `criterion` on either query defaults to `PreferenceCriterion.WEIGHTED_COUNT`, matching
  `best_worlds`'s own default.
- Both queries are pure functions of their inputs: identical inputs must produce identical boolean
  results, including across separate process runs (inherited from `best_worlds`/`preferred`'s own
  determinism guarantees — this story doesn't introduce new nondeterminism on top).

## Acceptance criteria

1. `is_obligatory(b, given, rules, hard_constraints, criterion)` returns `True` iff `b <= w` for every
   `w` in `best_worlds(given, rules, hard_constraints, criterion)`.
2. `is_permitted(b, given, rules, hard_constraints, criterion)` returns `True` iff `b <= w` for at least
   one `w` in `best_worlds(given, rules, hard_constraints, criterion)`.
3. When `best_worlds(given, ...)` is empty, `is_obligatory` returns `True` and `is_permitted` returns
   `False`.
4. `is_obligatory(frozenset(), given, rules)` is `True` whenever `best_worlds(given, ...)` is non-empty.
5. Omitting `criterion` on either function defaults to `PreferenceCriterion.WEIGHTED_COUNT`.
6. Both queries return identical results across repeated calls with identical inputs.

## Tasks

1. Create `src/deontic_reasoner/queries.py`.
2. Implement `is_obligatory` and `is_permitted` on top of `best_worlds`.
3. Write `tests/test_queries_basic.py` covering acceptance criteria 1–6.

## Deliverables

- `project/src/deontic_reasoner/queries.py` (new)
- `project/tests/test_queries_basic.py` (new)

## Dependencies

- `2-preference-ordering-and-best-worlds` (feature, all 3 stories) — needs `best_worlds`.
