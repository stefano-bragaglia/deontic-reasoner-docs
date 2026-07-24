# 2. DONE - Temporal Scope Evaluation

## What it does

`scope_matches(norm, request, registry, now) -> bool` — combines temporal-bound checking
(`valid_from`/`valid_until`) with `1-predicate-registry-and-condition-evaluation`'s condition check.
Direct implementation of adapted worked scenario §14.1: re-evaluated fresh at decision time, never
cached from when the norm was created, per `Requirements.md` FR 13.

## Inputs / outputs

```python
scope_matches(Norm(..., valid_until=t0 + timedelta(hours=24)), request={}, registry={}, now=t0 + timedelta(hours=1))
# -> True

scope_matches(Norm(..., valid_until=t0 + timedelta(hours=24)), request={}, registry={}, now=t0 + timedelta(hours=25))
# -> False
```

## Edge cases and failure modes

- `valid_from=None` means no lower bound; `valid_until=None` means no upper bound. A fully-default
  `Norm` (no temporal bounds, no condition) is in scope for any `now`/`request`.
- Boundary values are **inclusive** on both ends: `now == valid_from` and `now == valid_until` both
  count as in scope (with the condition, if any, also satisfied) — the same explicit convention adopted
  in an earlier round, since no source material pins this down either way.
- `scope_matches` requires **both** the temporal check and the condition check (via
  `evaluate_condition`) to pass — failing either makes the whole result `False`.
- **Re-evaluated fresh, not cached**: the same `Norm` evaluated at two different `now` values (one
  inside, one outside its temporal bound) must return different results within the same test run — the
  direct §14.1 regression, and the concrete proof that scope is checked at decision time rather than
  memoized from when the norm was created.

## Acceptance criteria

1. A fully-default `Norm` (no temporal bounds, no condition) is in scope for any `now`/`request`/empty
   `registry`.
2. `now < norm.valid_from` or `now > norm.valid_until` makes it out of scope; `now == valid_from` and
   `now == valid_until` (with the condition, if any, satisfied) are both in scope.
3. A norm within its temporal window but failing its named condition (or vice versa) is out of scope —
   `scope_matches` is `False` if either check fails.
4. The same `Norm` evaluated at two different `now` values (one inside, one outside its temporal
   bound) returns different results in the same test run — implementing the exact §14.1 setup
   (`valid_until = T+24h`, checked at `T+1h` and `T+25h`).

## Tasks

1. Add `scope_matches(norm, request, registry, now)` to `scope.py`.
2. Write `tests/test_scope_matches.py` covering acceptance criteria 1–4.

## Deliverables

- `project/src/deontic_reasoner/scope.py` (modified)
- `project/tests/test_scope_matches.py` (new)

## Dependencies

- `1-predicate-registry-and-condition-evaluation`.
