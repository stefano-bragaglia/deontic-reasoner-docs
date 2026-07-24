# 3. DONE - Chisholm's Paradox Regression

## What it does

Direct implementation of adapted worked scenario §14.8 — the historical regression test for the exact
failure mode dyadic obligation exists to prevent. Reproduces the Chisholm quartet: an unconditional
obligation, two dyadic obligations guarded by complementary (separately-named, not logically negated)
conditions, and a fact establishing which of the two actually applies. The reasoner must derive the one
matching the facts, not the other, and never a contradiction.

## Inputs / outputs

```python
rules = [
    Rule(body=frozenset(), head=frozenset({"request_approval"}), weight=5.0),
    Rule(body=frozenset({"requested"}), head=frozenset({"log_request"}), weight=5.0),
    Rule(body=frozenset({"not_requested"}), head=frozenset({"flag_unauthorized"}), weight=5.0),
]
given = frozenset({"not_requested"})   # approval was not, in fact, requested

is_obligatory({"flag_unauthorized"}, given, rules)   # -> True
is_obligatory({"log_request"}, given, rules)          # -> False
```

## Edge cases and failure modes

- The second rule's body (`requested`) is never forced true by anything in this scenario — since
  nothing in `given` or any other rule constrains the `requested` atom, `best_worlds(given, ...)`
  includes worlds on both sides of it, so `log_request` is **not** obligatory (it holds in some but not
  all best worlds — the rule guarding it simply never gets triggered, it doesn't get vacuously satisfied
  in a way that forces `log_request` either way).
- The third rule's body (`not_requested`) **is** in `given`, so it always applies — every best world
  must satisfy `flag_unauthorized` to avoid paying its weight, making it genuinely obligatory.
- No test in this story ever asserts both `flag_unauthorized` and `log_request` are simultaneously
  obligatory, and no code path here raises an exception or produces an inconsistent result — the two
  dyadic conditions never both fire simultaneously because they're guarded by complementary,
  separately-named atoms, exactly the design choice (no true logical negation on `Atom`) already
  established for this framework.
- This test exists specifically to catch a regression if `violates`/`best_worlds`'s handling of
  conditional rules is ever carelessly changed to something resembling material-conditional semantics
  (vacuously true when the body is false, licensing bad inferences) instead of the current, correct
  "the rule simply doesn't apply" behavior.

## Acceptance criteria

1. With the three-rule set and `given = {"not_requested"}` above, `is_obligatory({"flag_unauthorized"}, given, rules)`
   is `True`.
2. With the same setup, `is_obligatory({"log_request"}, given, rules)` is `False`.
3. No exception is raised and no test in this story asserts both obligations hold simultaneously.

## Tasks

1. Write `tests/test_queries_chisholm_paradox.py` implementing the scenario and both queries above.

## Deliverables

- `project/tests/test_queries_chisholm_paradox.py` (new)

## Dependencies

- `1-obligation-and-permissibility-queries`.
