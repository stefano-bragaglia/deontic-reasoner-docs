# 4. Condition and Scope Evaluation

## Name

`4-condition-and-scope-evaluation`

## Summary

How a `Norm`'s `Scope` (temporal bounds, execution contexts, registered conditions) and `Condition`
guards get evaluated against a concrete request/context — safely. `Condition` objects are resolved
through a fixed, engine-owned predicate registry (`{"violation": ..., "business_hours": ...}`); this
feature exists specifically to make it impossible to reach for `eval()`/`exec()` on a condition that
ultimately traces back to agent-supplied data, which would be arbitrary code execution. Scope filtering
must be evaluated at *decision* time, not cached from derivation time, since a norm can expire between
being derived and being queried — this feature owns that re-check, so every later consumer (the
resolution pipeline) gets it for free rather than re-implementing it per call site.

## Requirements covered

- FR 6 — scoped norms evaluated at decision time, not only derivation time.
- FR 7 — condition evaluation only through a fixed predicate registry, never `eval()`/`exec()`.
- NFR 2 / NFR 3 (risk R-7 in the implementation spec) — no dynamic code execution on agent-supplied data.
- Acceptance criteria: implementation spec worked scenario §14.1 (privilege with scope expiry — the
  scope filter must correctly exclude an expired norm at decision time and fall through to closure
  policy).

## Dependencies

- `1-core-data-model` — `Scope`, `Condition` must exist. Does not depend on the engine
  (`2-forward-chaining-engine`) or the domain rules (`3-hohfeldian-and-delegation-semantics`) — scope/
  condition evaluation is a pure function of a `Norm` and a request context, usable independently of
  forward chaining.

## Stories

See `documentation/features/4-condition-and-scope-evaluation/` for the story breakdown (added by
`/stories`).
