# 2. DONE - Weighted-Conflict Resolution Regression

## What it does

Direct implementation of adapted worked scenario §14.4: two rules of different weight jointly express
"an action is allowed by default, but forbidden under some condition" — a default low-weight rule
permitting it, and a higher-weight conditional rule prohibiting it. Confirms `is_permitted` resolves the
conflict correctly by weighted-count preference, with **no** separate conflict-explanation object — the
explicit capability the superseded SAT/unsat-core layer provided and this design deliberately drops
(Requirements.md → Questions: Scenario Coverage #5).

## Inputs / outputs

Two rules over distinct, domain-named atoms (negation is a separate named atom, not a logical operator,
consistent with how `not_requested` was already handled in the Chisholm scenario):

```python
default_send = Rule(body=frozenset(), head=frozenset({"sent"}), weight=1.0)
external_prohibition = Rule(body=frozenset({"external_recipient"}),
                             head=frozenset({"not_sent"}), weight=10.0)   # weighted higher: wins

is_permitted({"sent"}, given=frozenset(), rules=[default_send, external_prohibition])
# -> True   (external_prohibition's body is false, so it never applies)

is_permitted({"sent"}, given=frozenset({"external_recipient"}), rules=[default_send, external_prohibition])
# -> False  (external_prohibition applies and outweighs default_send)
```

## Edge cases and failure modes

- `external_prohibition`'s body being false (internal-recipient case) means it never applies at all —
  the query result in that case depends only on `default_send`.
- The exact weight values only need to establish the *ordering* (prohibition weighted higher than the
  default) — this story doesn't pin down specific numbers beyond "prohibition's weight is high enough to
  dominate the comparison," leaving the precise values to Stage A's actual test.
- Nothing in this test constructs or asserts a conflict-explanation object (an unsat core, a "which
  norms conflict" result, etc.) — only the two query outcomes above are asserted, since that
  explanatory capability doesn't exist in this design.

## Acceptance criteria

1. With no `external_recipient` fact in `given`, `is_permitted({"sent"}, given, rules)` is `True`.
2. With `external_recipient` in `given`, `is_permitted({"sent"}, given, rules)` is `False` — the
   higher-weight prohibition rule dominates the weighted-count comparison.
3. No conflict-explanation object of any kind is asserted in this test — only the two boolean query
   results.

## Tasks

1. Write `tests/test_queries_weighted_conflict.py` implementing the scenario above, using
   `is_permitted` from `1-obligation-and-permissibility-queries`. No new production code is expected; if
   a real gap surfaces, extend `queries.py` minimally.

## Deliverables

- `project/tests/test_queries_weighted_conflict.py` (new)

## Dependencies

- `1-obligation-and-permissibility-queries`.
