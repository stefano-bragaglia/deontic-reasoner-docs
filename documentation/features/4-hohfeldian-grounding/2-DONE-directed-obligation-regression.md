# 2. DONE - Directed Obligation Regression

## What it does

Direct implementation of adapted worked scenario §14.2: end-to-end proof that grounding a `RIGHT` norm
produces a genuinely **directed** obligation — the *counterparty's* duty is obligatory once the norm
holds — verified via `is_obligatory` from `3-obligation-and-permissibility-queries`. This is the payoff
that justifies this feature's existence: plain `O`/dyadic-`O(q|p)` alone cannot express *who* owes a duty
to *whom*; grounding plus the query engine together can.

## Inputs / outputs

```python
norm = Norm(id="n1", relation=Relation.RIGHT, subject="FileAgent", action="write",
            resource="/out/report.csv", counterparty="filesystem")
atom, rule = ground_norm(norm)

is_obligatory({duty_atom}, given=frozenset({atom}), rules=[rule])   # -> True
is_obligatory({duty_atom}, given=frozenset(), rules=[rule])          # -> False
```

## Edge cases and failure modes

- The duty atom must be parameterized by the **counterparty** (`"filesystem"`), never by the norm's own
  `subject` (`"FileAgent"`) — directedness specifically means the obligation falls on the counterparty,
  not reflexively on the right-holder. This test asserts the duty atom's identity differs from what a
  subject-parameterized version would produce, not just that *some* obligation was derived.
- Without the norm's own atom present in `given`, the correlative rule's body is unsatisfied, so
  `best_worlds` doesn't force the duty atom either way — the duty must be genuinely **conditional** on
  the norm holding, not a standing fact that's obligatory regardless.

## Acceptance criteria

1. With the norm's own atom in `given` and the correlative rule in `rules`, `is_obligatory` for the
   counterparty's duty atom is `True`.
2. With an empty (or otherwise norm-atom-free) `given`, the same query is `False`.
3. The duty atom's string content is built from the counterparty, not the norm's subject — verified by
   comparing against what a (deliberately wrong) subject-parameterized atom name would look like.

## Tasks

1. Write `tests/test_grounding_directed_obligation.py` implementing the scenario above, using
   `ground_norm` (`1-norm-atom-and-correlative-rule-grounding`) and `is_obligatory`
   (`3-obligation-and-permissibility-queries`, story 1).

## Deliverables

- `project/tests/test_grounding_directed_obligation.py` (new)

## Dependencies

- `1-norm-atom-and-correlative-rule-grounding`.
- `3-obligation-and-permissibility-queries` (feature, story 1) — needs `is_obligatory`.
