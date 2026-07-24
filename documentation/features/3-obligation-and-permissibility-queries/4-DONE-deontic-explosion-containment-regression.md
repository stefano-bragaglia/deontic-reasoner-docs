# 4. DONE - Deontic-Explosion Containment Regression

## What it does

Direct implementation of adapted worked scenario §14.9. Confirms that an unrelated query's answer is
unaffected by a conflicting, unrelated rule pair elsewhere in the same rule set — under this framework
this containment is a **structural** consequence of atom-disjointness (weighted-count cost is additive
across atom-disjoint rule groups), not the result of a deliberate partitioning design decision the
superseded SAT layer needed (Requirements.md → Questions: Scenario Coverage #5).

## Inputs / outputs

```python
conflicting_pair = [
    Rule(body=frozenset(), head=frozenset({"task1"}), weight=3.0),
    Rule(body=frozenset(), head=frozenset({"not_task1"}), weight=7.0),
]
unrelated = Rule(body=frozenset(), head=frozenset({"task2"}), weight=4.0)

is_obligatory({"task2"}, given=frozenset(), rules=[*conflicting_pair, unrelated])   # -> True
```

## Edge cases and failure modes

- `task1`/`not_task1` share no atoms with `task2` at all — because `WEIGHTED_COUNT`'s total violated
  weight is additive across independent rules, minimizing the `task2`-related term is entirely separable
  from however the `task1`/`not_task1` tension resolves; there's no shared-atom coupling that could let
  one group's conflict leak into the other's outcome.
- The result must hold **regardless of which way the `task1`/`not_task1` conflict is weighted** — this
  story's test swaps which of the two has the higher weight and confirms `task2`'s obligation status is
  unchanged either way, proving the independence isn't an accident of one particular weight choice.
- There is no "scope this check to a subject/resource" parameter anywhere in this feature's API (unlike
  the superseded SAT design's explicit per-`(subject, resource)` partitioning) — this test's containment
  comes from ordinary weighted-count arithmetic over disjoint atoms, not from any explicit isolation
  mechanism in the production code.

## Acceptance criteria

1. With the conflicting pair and the unrelated rule combined into one `rules` list,
   `is_obligatory({"task2"}, given=frozenset(), rules)` is `True`.
2. The result in criterion 1 is unchanged when the conflicting pair's weights are swapped (whichever of
   `task1`/`not_task1` "wins" internally).
3. No function call anywhere in this test passes a subject/resource-scoping parameter — the containment
   demonstrated is structural, not the result of explicit partitioning.

## Tasks

1. Write `tests/test_queries_deontic_explosion.py` implementing the scenario above, including the
   weight-reversal check (criterion 2).

## Deliverables

- `project/tests/test_queries_deontic_explosion.py` (new)

## Dependencies

- `1-obligation-and-permissibility-queries`.
