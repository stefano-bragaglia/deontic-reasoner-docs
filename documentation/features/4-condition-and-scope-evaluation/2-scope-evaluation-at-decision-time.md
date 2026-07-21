# 2. Scope Evaluation at Decision Time

## What it does

`scope_is_active(scope, at, context, registry)` evaluates a `Scope`'s temporal bounds, execution-context
membership, and all of its `conditions` (via `1-predicate-registry`'s `evaluate_condition`) against a
concrete decision-time snapshot. It is re-evaluated fresh on every call — never cached from when the
norm was derived — since a norm can expire between derivation and use (Requirements.md FR 6).

## Inputs / outputs

```python
scope_is_active(
    Scope(valid_until=t0 + timedelta(hours=24)),
    at=t0 + timedelta(hours=1), context=None, registry={},
) # -> True

scope_is_active(
    Scope(valid_until=t0 + timedelta(hours=24)),
    at=t0 + timedelta(hours=25), context=None, registry={},
) # -> False
```

## Edge cases and failure modes

- `valid_from=None` means no lower bound; `valid_until=None` means no upper bound. A fully-default
  `Scope()` is active for any `at`/`context`.
- Boundary values are **inclusive** on both ends: `at == valid_from` and `at == valid_until` both count
  as active (nothing in the source material pins this down explicitly, so this story fixes it
  explicitly rather than leaving it ambiguous for Stage B to guess at).
- `scope.contexts` empty (the default) means **unrestricted** — matches any `context`, including
  `context=None`. `scope.contexts` non-empty means `context` must be a member; `context=None` against a
  non-empty `contexts` set is **not** a member and fails.
- `scope.conditions` empty means vacuously satisfied (no extra guards). Non-empty means **all** must
  evaluate `True` via `evaluate_condition` (conjunction) — one `False` makes the whole scope inactive.
- An `UnknownPredicateError` raised while evaluating one of `scope.conditions` propagates unchanged, it
  is not caught/treated as `False` here.

## Acceptance criteria

1. A fully-default `Scope()` is active for any `at`/`context`/empty registry.
2. `at < scope.valid_from` or `at > scope.valid_until` makes the scope inactive; `at == valid_from` and
   `at == valid_until` (with everything else satisfied) are both active.
3. Non-empty `scope.contexts` with `context` not a member (including `context=None`) makes the scope
   inactive.
4. Empty `scope.contexts` is active regardless of `context`.
5. Any single failing `Condition` in `scope.conditions` makes the whole scope inactive; all passing makes
   it active.
6. The same `Scope` evaluated at two different `at` values (one inside, one outside its temporal bound)
   returns different results in the same test run — the direct regression check for §14.1 (a scope that
   was valid an hour after grant is no longer valid a day after grant, evaluated fresh each time, not
   memoized from derivation).

## Tasks

1. Add `scope_is_active(scope, at, context, registry)` to `src/deontic_reasoner/conditions.py`.
2. Write `tests/test_scope_evaluation.py` covering acceptance criteria 1–6, including the exact §14.1
   setup (`valid_until = T+24h`, checked at `T+1h` and `T+25h`).

## Deliverables

- `project/src/deontic_reasoner/conditions.py` (modified)
- `project/tests/test_scope_evaluation.py` (new)

## Dependencies

- `1-predicate-registry` (this feature).
- `1-core-data-model` (feature) — needs `Scope`.
