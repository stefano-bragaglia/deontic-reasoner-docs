# 5. DONE - Scope Evaluation

## Name

`5-scope-evaluation`

## Summary

`scope_matches(norm, request)` — temporal/context/condition applicability, so an expired or
out-of-context norm doesn't apply to a given request. Resolved only through a fixed, engine-owned
predicate registry (`(predicate_name, args)` pairs mapped to Python callables) — never
`eval()`/`exec()`/`ast.literal_eval` on data that traces back to an external agent request, since a
norm's condition data can ultimately originate from an untrusted request. This carries over the same
trust-boundary requirement considered in an earlier round, unchanged by the dyadic/preferential pivot
(Requirements.md → Questions: Scope Evaluation #3).

## Requirements covered

- FR 13 — `scope_matches(norm, request) -> bool`.
- NFR 3 — no dynamic code execution on externally-supplied condition data.
- Acceptance criteria: §14.1 — a norm's scope excludes it once its temporal bound has passed, re-checked
  fresh at decision time rather than cached from when the norm was created.

## Dependencies

- `1-core-data-model` — needs `Norm`.

## Stories

See `documentation/features/5-scope-evaluation/` for the story breakdown (added by `/stories`).
