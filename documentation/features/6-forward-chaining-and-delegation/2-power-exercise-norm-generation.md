# 2. Power-Exercise Norm Generation

## What it does

`exercise_power(power_norm, subject, relation, action, resource, counterparty=None, valid_from=None,
valid_until=None, condition=None) -> Norm` — represents a Hohfeldian **power's exercise**: given the
norm granting the power, constructs the new `Norm` it brings into existence. No dedicated predicate or
special mechanism is needed (per `Requirements.md` → Questions: Power and Immunity Semantics #6) — this
is an ordinary, deterministic function, generalizing the same "derive from a fact" idea
`1-delegation-obligations-grounding` already uses for delegation specifically.

## Inputs / outputs

```python
power_norm = Norm(id="p1", relation=Relation.POWER, subject="Orchestrator",
                   action="grant", resource="SubAgent.permissions", counterparty="SubAgent")

new_norm = exercise_power(power_norm, subject="SubAgent", relation=Relation.PRIVILEGE,
                           action="read", resource="R2")
# new_norm.id is a deterministic function of (power_norm.id, subject, relation, action, resource, counterparty)
```

## Edge cases and failure modes

- **Determinism**: calling `exercise_power` twice with identical arguments produces two *equal* `Norm`s
  (same `id`, same every field) — required so that feeding the result into
  `4-forward-chaining-fixed-point-loop` twice is a no-op for convergence purposes, not a growing set of
  duplicate norms.
- This function does **not** validate that `power_norm.relation == Relation.POWER` — consistent with
  the established convention that relation-specific invariants aren't enforced at the data/function
  level (`1-core-data-model` → `2-hohfeldian-norm-and-relation`'s own note). It's a plain constructor,
  not a policy check.
- This iteration's `Norm` has no `provenance`/`granted_by` field at all (unlike an earlier, now-
  superseded design) — the derived norm does not record which power exercise produced it beyond
  whatever the caller encodes in its own fields (e.g. `condition`). Traceability, if ever needed, is
  later-iteration work.
- **§14.6, immunity half**: an `IMMUNITY` norm (e.g. protecting `AuditAgent`'s audit-log-write access
  from revocation) can be constructed and passed to `4-hohfeldian-grounding`'s `ground_norm` without
  error, producing its own atom and disability rule — purely representational, per `Requirements.md` →
  Questions: Power and Immunity Semantics #7. `exercise_power` called to derive a *revoking* norm over
  the same resource **succeeds unconditionally** — it does not check for, and is not blocked by, the
  immunity norm's existence. This test exists specifically to document the current, deliberate absence
  of short-circuit behavior, not to accidentally introduce it.

## Acceptance criteria

1. `exercise_power(power_norm, subject, relation, action, resource, counterparty=None, ...)` returns a
   `Norm` with exactly those fields, and an `id` deterministic in `(power_norm.id, subject, relation,
   action, resource, counterparty)`.
2. Calling it twice with identical arguments returns two equal `Norm`s.
3. The returned `Norm` is valid input to `ground_norm` (`4-hohfeldian-grounding`) — e.g. with
   `relation=PRIVILEGE`, `ground_norm` returns `(atom, None)` as normal; this is `§14.6`'s
   power-creates-a-norm half.
4. An `IMMUNITY` norm passed to `ground_norm` produces its own atom and disability rule without error.
5. `exercise_power` succeeds (no exception, a normal `Norm` returned) when constructing a revoking norm
   over a resource an `IMMUNITY` norm elsewhere protects — demonstrating no short-circuit exists yet.

## Tasks

1. Add `exercise_power` to `src/deontic_reasoner/delegation.py`.
2. Write `tests/test_power_exercise.py` covering acceptance criteria 1–5.

## Deliverables

- `project/src/deontic_reasoner/delegation.py` (modified)
- `project/tests/test_power_exercise.py` (new)

## Dependencies

- `1-core-data-model` (feature) — needs `Norm`, `Relation`.
- `4-hohfeldian-grounding` (feature, story 1) — needs `ground_norm` for criteria 3–5.
