# 3. Dyadic Norm Activation

## What it does

`norm_is_active(norm, registry)` extends condition evaluation to a `Norm`'s own `given` field: an
unconditional norm (`given=None`) is always active; a dyadic norm (`given=Condition(...)`) is active
only when its condition currently evaluates true via `1-predicate-registry`'s `evaluate_condition`. This
is the piece that makes `3-hohfeldian-and-delegation-semantics`'s delegation-derived
revoke-on-violation obligation actually mean something at decision time — and the piece that
feature 3 explicitly deferred for full Chisholm's-paradox coverage (§14.8).

## Inputs / outputs

```python
revoke_norm = Norm(..., given=Condition("violation", ("VisualizationAgent", "p1")))
norm_is_active(revoke_norm, {"violation": lambda args: args in violated_set})
# -> True only if ("VisualizationAgent", "p1") is currently a known violation.
```

## Edge cases and failure modes

- `norm.given is None` → always `True` (unconditional), without ever calling the registry.
- `norm.given` set → returns exactly `evaluate_condition(norm.given, registry)`'s result; no additional
  logic layered on top.
- **§14.8 regression, reproduced directly**: given a registry where a `"requested"` predicate evaluates
  `False` and a `"not_requested"` predicate evaluates `True` for the relevant args, a norm with
  `given=Condition("requested", ...)` is inactive and a **separate** norm with
  `given=Condition("not_requested", ...)` is active — both derived from the same registry/context with
  no code path that asserts both, or neither, are active. There is no logical negation operator on
  `Condition` — "not requested" is simply its own registered predicate name, exactly as the underlying
  facts of the scenario are themselves separate ground propositions, not one proposition and its
  negation.
- Pure function: no side effects, safe to call repeatedly with the same inputs and get the same result
  (no hidden caching, no mutation of `norm` or `registry`).

## Acceptance criteria

1. `norm_is_active(norm, registry)` returns `True` when `norm.given is None`, without invoking `registry`.
2. Returns `evaluate_condition(norm.given, registry)`'s result when `norm.given` is set.
3. §14.8 end-to-end: constructing the four elements of the worked scenario (an `O(request_approval)`
   norm, a dyadic `O(log_request | requested)` norm, a dyadic `O(flag_unauthorized | ¬requested)` norm,
   and a registry reflecting "did not request") — `norm_is_active` reports the `flag_unauthorized` norm
   active and the `log_request` norm inactive, with no contradiction derived anywhere in the process.
4. Calling `norm_is_active` twice with identical inputs returns identical results.

## Tasks

1. Add `norm_is_active(norm, registry)` to `src/deontic_reasoner/conditions.py`.
2. Write `tests/test_norm_activation.py` covering acceptance criteria 1–4, with criterion 3 built as a
   direct, literal reproduction of implementation-spec worked scenario §14.8.

## Deliverables

- `project/src/deontic_reasoner/conditions.py` (modified)
- `project/tests/test_norm_activation.py` (new)

## Dependencies

- `1-predicate-registry` (this feature) — only needs `evaluate_condition`, not
  `2-scope-evaluation-at-decision-time`'s scope logic.
- `1-core-data-model` (feature) — needs `Norm`, `Condition`.
