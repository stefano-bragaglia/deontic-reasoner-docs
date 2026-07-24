# 3. DONE - Best-Worlds Computation with Scoped Enumeration

## What it does

`best_worlds(antecedent, rules, hard_constraints=(), criterion=PreferenceCriterion.WEIGHTED_COUNT) ->
set[World]` — the semantic core both queries in `3-obligation-and-permissibility-queries` run against.
Enumerates candidate worlds scoped to the atoms actually mentioned in `antecedent`/`rules` (not the full
global atom vocabulary, per `Requirements.md` → Questions: Data Representation #1), excludes any world a
hard constraint rules out, filters to those satisfying `antecedent`, and returns the most-preferred
remainder.

## Inputs / outputs

```python
best_worlds(antecedent=frozenset({"delegated"}), rules=[...], hard_constraints=[...])
# -> {World({"delegated", "audit"}), ...}
```

## Edge cases and failure modes

- **Scoped enumeration**: only atoms appearing in `antecedent` or in any rule's `body`/`head` are
  varied; every other atom in the system is marginalized out entirely (never appears true or false in
  any candidate world, since it can't affect which world is preferred). Enumeration size is
  `2^k` where `k` is the count of *relevant* atoms not already fixed by `antecedent`, not the system's
  total atom count.
- `antecedent` atoms are fixed **true** in every candidate world (that's what "satisfying the
  antecedent" means) — only the relevant atoms *not* in `antecedent` are varied across combinations.
- Hard constraints are applied **before** preference comparison — a world violating any
  `HardConstraint` is excluded from the candidate set entirely, even if it would otherwise be the single
  most-preferred world.
- **Ties**: when multiple worlds tie for most-preferred under the given criterion, `best_worlds` returns
  **all** of them (a `set`), not an arbitrary single pick — this matters directly for the permissibility
  query (`3-obligation-and-permissibility-queries`), which asks whether something holds in *some*
  best world.
- **Vacuous case**: if every world satisfying `antecedent` is excluded by a hard constraint,
  `best_worlds` returns an **empty set** — this is valid, documented behavior, not an error. (What an
  empty result means for the obligation/permissibility queries built on top of it is
  `3-obligation-and-permissibility-queries`'s decision to make, not this story's.)
- Default `criterion` is `PreferenceCriterion.WEIGHTED_COUNT` when the caller omits it.

## Acceptance criteria

1. Every world in the returned set has every atom in `antecedent` true.
2. Enumeration is scoped to atoms in `antecedent` or any rule's `body`/`head` — an atom mentioned in
   neither has no effect on the returned set (adding an irrelevant, unmentioned atom name anywhere in
   the call changes nothing).
3. A world violating any `HardConstraint` in `hard_constraints` is never included in the result, even
   when it would otherwise be uniquely most-preferred.
4. When multiple worlds tie for most-preferred, `best_worlds` returns all of them.
5. When every antecedent-satisfying world is hard-constraint-excluded, `best_worlds` returns an empty
   set (verified as a normal return, not a raised exception).
6. Omitting `criterion` defaults to `PreferenceCriterion.WEIGHTED_COUNT`.

## Tasks

1. Add `best_worlds(antecedent, rules, hard_constraints=(), criterion=PreferenceCriterion.WEIGHTED_COUNT)`
   to `preference.py`: compute the relevant atom universe, enumerate candidates with `antecedent` fixed
   true and the rest varied, drop hard-constraint-excluded worlds, keep only the most-preferred
   remainder via `preferred`.
2. Write `tests/test_preference_best_worlds.py` covering acceptance criteria 1–6.

## Deliverables

- `project/src/deontic_reasoner/preference.py` (modified)
- `project/tests/test_preference_best_worlds.py` (new)

## Dependencies

- `1-rule-violation-and-hard-constraint-exclusion`, `2-preference-ordering-three-criteria`.
