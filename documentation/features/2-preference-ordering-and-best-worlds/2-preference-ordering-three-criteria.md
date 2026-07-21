# 2. Preference Ordering — Three Criteria

## What it does

`violated_rules(world, rules) -> frozenset[Rule]` and `preferred(world_a, world_b, rules, criterion)
-> bool` under one of three configurable criteria — **subset/Pareto**, **count**, or **weighted count**
(the default) — over each world's violated-rule set. This is what directly replaces the previous
(superseded) design's SAT-based conflict detection and combining algorithms: which world "wins" falls
out of this comparison, no separate machinery needed.

## Inputs / outputs

```python
violated_rules(world, rules) -> frozenset[Rule]   # exactly the rules `violates(rule, world)` holds for

preferred(world_a, world_b, rules, PreferenceCriterion.WEIGHTED_COUNT)  # -> bool
```

## Edge cases and failure modes

- **`SUBSET`**: `a` is preferred over `b` iff `violated_rules(a) <= violated_rules(b)` (a genuine subset
  relation). This criterion can leave two worlds **incomparable**: a case must exist where
  `preferred(a, b, ..., SUBSET)` and `preferred(b, a, ..., SUBSET)` are **both** `False` — incomparability
  is not a tie, and this criterion must not silently pretend otherwise.
- **`COUNT`**: `a` is preferred over `b` iff `len(violated_rules(a)) <= len(violated_rules(b))` — always
  comparable (a total order); equal counts make both directions `True` (equally preferred).
- **`WEIGHTED_COUNT`** (the default when no criterion is given): `a` is preferred over `b` iff the
  summed `weight` of `a`'s violated rules is `<=` that of `b`'s — also always comparable, with equal sums
  making both directions `True`.
- **Determinism across process runs, not just within one** (`Requirements.md` NFR 4): Python randomizes
  string hashing per-process by default, so a `frozenset[Rule]`'s *iteration order* can differ between
  separate interpreter invocations even for the identical set of rules. Summing `weight` values in
  raw iteration order would make `WEIGHTED_COUNT` vulnerable to floating-point rounding differing
  between runs (float addition isn't associative), which would violate "identical inputs produce
  identical outputs on every run." The summation must therefore iterate in a **stable, sorted order** —
  e.g. sorted by `(sorted(rule.body), sorted(rule.head), rule.weight)`, which depends only on string
  comparison (`<`), never on hash values — not raw `frozenset` iteration order.
- An empty `rules` list means every world violates zero rules, so every pair of worlds is equally
  preferred under all three criteria.
- A rule whose body is false in both `a` and `b` contributes to neither's violated set and so never
  affects the comparison.

## Acceptance criteria

1. `PreferenceCriterion` is an enum with `SUBSET`, `COUNT`, `WEIGHTED_COUNT`.
2. `violated_rules(world, rules)` returns exactly the subset of `rules` for which `violates(rule, world)`
   is `True`.
3. Under `SUBSET`, `preferred(a, b, rules, SUBSET)` is `True` iff `violated_rules(a, rules)` is a subset
   of `violated_rules(b, rules)`.
4. A constructed case exists where `SUBSET` reports **both** `preferred(a, b, ...)` and
   `preferred(b, a, ...)` as `False` (genuinely incomparable worlds).
5. Under `COUNT`, `preferred(a, b, rules, COUNT)` is `True` iff `len(violated_rules(a, rules)) <=
   len(violated_rules(b, rules))`; for any pair of worlds, at least one direction is always `True`.
6. Under `WEIGHTED_COUNT` (also the default when `criterion` is omitted), `preferred(a, b, rules,
   WEIGHTED_COUNT)` is `True` iff `a`'s violated-rule weight sum is `<=` `b`'s; at least one direction is
   always `True`.
7. `WEIGHTED_COUNT`'s weight summation iterates in a stable, sort-key-based order (not raw `frozenset`
   iteration order) — verified indirectly by asserting the sort key used depends only on `body`/`head`
   string content, never on `hash()`/iteration order of a `set`/`frozenset`.
8. With an empty `rules` list, every pair of worlds is equally preferred (`preferred` is `True` in both
   directions) under all three criteria.

## Tasks

1. Add `PreferenceCriterion`, `violated_rules`, `preferred` to `preference.py`.
2. Implement `WEIGHTED_COUNT`'s summation with the stable sort key described above.
3. Write `tests/test_preference_ordering.py` covering acceptance criteria 1–8.

## Deliverables

- `project/src/deontic_reasoner/preference.py` (modified)
- `project/tests/test_preference_ordering.py` (new)

## Dependencies

- `1-rule-violation-and-hard-constraint-exclusion` (needs `violates`).
