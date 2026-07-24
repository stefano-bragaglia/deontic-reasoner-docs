# 1. DONE - Predicate Registry and Condition Evaluation

## What it does

The engine-owned mapping from a condition name to a plain Python callable, and the function that
resolves a `Norm`'s `condition` field through it — the only path by which a condition is ever evaluated.
This makes it structurally impossible to reach for `eval()`/`exec()` on data that traces back to an
external agent request (`Requirements.md` NFR 3), the same trust-boundary guarantee considered in an
earlier round, carried over unchanged (Requirements.md → Questions: Scope Evaluation #3).

## Inputs / outputs

```python
PredicateRegistry = Mapping[str, Callable[[Norm, Mapping[str, object]], bool]]

registry = {"business_hours": lambda norm, request: 9 <= request["hour"] < 17}
evaluate_condition(norm, request, registry)   # -> bool
```

`request` is a plain `Mapping[str, object]` — no dedicated dataclass; a registered callable receives
both the `Norm` and the request context so it has everything it needs without a separate `args` tuple.

## Edge cases and failure modes

- `norm.condition is None` → `evaluate_condition` returns `True` unconditionally, without consulting
  the registry — no extra guard beyond temporal bounds.
- An unregistered condition name is a **configuration bug**, not "the condition doesn't hold" — it must
  raise a specific `UnknownPredicateError`, never silently default to `True`/`False`.
- No code path here may call `eval`, `exec`, or `ast.literal_eval` on `norm.condition` — verified with a
  condition name that looks like Python syntax (e.g. `"__import__('os').system('echo pwned')"`), which
  must be treated as an ordinary (unregistered) string and raise `UnknownPredicateError` like any other
  unknown name, never executed.
- Lookup is exact-string match — a typo'd condition name is `UnknownPredicateError`, never a fuzzy
  fallback.

## Acceptance criteria

1. `evaluate_condition(norm, request, registry)` returns `True` when `norm.condition is None`, without
   invoking `registry`.
2. Returns `registry[norm.condition](norm, request)`'s result when `norm.condition` is set and present
   in `registry`.
3. Raises `UnknownPredicateError` when `norm.condition` is set but absent from `registry`.
4. A condition name containing Python syntax is never executed — it raises `UnknownPredicateError` like
   any other unregistered name, verified by confirming no side effect from the embedded syntax occurs.

## Tasks

1. Create `src/deontic_reasoner/scope.py`.
2. Define `PredicateRegistry`, `UnknownPredicateError`, `evaluate_condition`.
3. Write `tests/test_scope_condition.py` covering acceptance criteria 1–4.

## Deliverables

- `project/src/deontic_reasoner/scope.py` (new)
- `project/tests/test_scope_condition.py` (new)

## Dependencies

- `1-core-data-model` (feature) — needs `Norm`.
