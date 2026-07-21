# 1. Predicate Registry

## What it does

The engine-owned mapping from predicate name to a plain Python callable, and the single function that
evaluates any `Condition` through it — the only path by which a `Condition`/`given` guard is ever
resolved. This exists specifically to make it structurally impossible to reach for `eval()`/`exec()` on
data that traces back to an external agent request (Requirements.md NFR 3 / implementation spec R-7).

## Inputs / outputs

- `Predicate = Callable[[tuple[str, ...]], bool]`.
- `PredicateRegistry = Mapping[str, Predicate]`, e.g. `{"violation": lambda args: ..., "business_hours": lambda args: ...}`.
- `evaluate_condition(condition: Condition, registry: PredicateRegistry) -> bool` — looks up
  `registry[condition.predicate]` and calls it with `condition.args`.

## Edge cases and failure modes

- An unregistered predicate name is a **configuration bug**, not "the condition doesn't hold" — it must
  raise a specific `UnknownPredicateError`, never silently default to `True` or `False`. Guessing here
  would be exactly the kind of silent, safety-relevant default this vault's earlier Requirements answers
  (default combining algorithm, default closure policy) deliberately avoided.
- Lookup is exact-string match — a typo'd predicate name is an `UnknownPredicateError`, never a silent
  no-match or fuzzy fallback.
- No code path here may call `eval`, `exec`, or `ast.literal_eval` on `condition.predicate` or
  `condition.args` — verified with a predicate name that looks like Python syntax (e.g.
  `"__import__('os').system('echo pwned')"`), which must be treated as an ordinary (unregistered) string
  and raise `UnknownPredicateError` like any other unknown name, never executed.
- A predicate function raising its own exception (a bug in the registered callable, not in this module)
  propagates unchanged — this module doesn't swallow or wrap predicate-internal errors.

## Acceptance criteria

1. `Predicate` and `PredicateRegistry` type aliases are defined.
2. `evaluate_condition(condition, registry)` returns `registry[condition.predicate](condition.args)`.
3. An unregistered predicate name raises `UnknownPredicateError` (a new, specific exception type), not a
   silent default.
4. A predicate name containing Python syntax is never executed — it's treated as an ordinary string key
   and raises `UnknownPredicateError` when unregistered.
5. Two conditions with the same predicate name but different `args` correctly pass their own `args`
   through to the registered callable (no accidental sharing/caching of arguments across calls).

## Tasks

1. Create `src/deontic_reasoner/conditions.py`.
2. Define `Predicate`, `PredicateRegistry`, `UnknownPredicateError`.
3. Implement `evaluate_condition(condition, registry)`.
4. Write `tests/test_conditions_registry.py` covering acceptance criteria 2–5.

## Deliverables

- `project/src/deontic_reasoner/conditions.py` (new)
- `project/tests/test_conditions_registry.py` (new)

## Dependencies

- `1-core-data-model` (feature) — needs `Condition`.
