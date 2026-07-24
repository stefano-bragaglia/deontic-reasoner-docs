# 3. DONE - Delegation Grant with Scope-Narrowing

## What it does

`delegate_with_narrowing(parent, subject, valid_from=None, valid_until=None, counterparty=None,
condition=None) -> Norm` — given a delegator's own norm (`parent`) and a candidate temporal window that
may be *wider* than what `parent` actually holds, derives a new norm for the delegatee whose
`valid_from`/`valid_until` are automatically narrowed to the intersection — the delegatee can never end
up with a wider window than the delegator held, per `Requirements.md` → Questions: Built-in Semantics
#4 (no opt-in required). This adapts §14.7's original context-*set* framing to this design's simpler,
temporal-only `Norm` scope fields (`1-core-data-model` has no `contexts` field) — documented here as a
deliberate scenario adaptation, not an oversight.

## Inputs / outputs

```python
parent = Norm(id="p1", relation=Relation.PRIVILEGE, subject="AnalyticsAgent",
              action="read", resource="/data/analytics/**",
              valid_from=t0, valid_until=t0 + timedelta(hours=24))

granted = delegate_with_narrowing(parent, subject="VisualizationAgent",
                                   valid_from=t0 - timedelta(hours=1), valid_until=t0 + timedelta(hours=48))
# granted.valid_from == t0                       (later of the two — narrowed)
# granted.valid_until == t0 + timedelta(hours=24) (earlier of the two — narrowed)
# granted.relation/action/resource == parent's, unchanged
```

## Edge cases and failure modes

- `None` bounds mean unbounded on that side; intersecting `None` with a concrete bound always yields
  the concrete bound (a missing bound never *widens* the result).
- A candidate window **narrower** than `parent`'s passes through unchanged (intersecting a subset with
  its superset is the subset itself) — not a separate code path, just the correct behavior of the same
  intersection logic.
- `relation`, `action`, and `resource` always carry over from `parent` unchanged — delegation transfers
  *who holds* the permission and narrows *when*, it never changes *what* the permission is about.
- The returned norm's `id` is a deterministic function of `(parent.id, subject, valid_from, valid_until,
  counterparty, condition)` — re-delegating with identical arguments never produces a second,
  differently-`id`'d grant.

## Acceptance criteria

1. `delegate_with_narrowing(parent, subject, ...)` returns a `Norm` whose `relation`/`action`/`resource`
   match `parent`'s exactly.
2. Given a candidate window wider than `parent`'s, the returned norm's `valid_from`/`valid_until` are
   the intersection — never wider than `parent`'s own window. This is the direct §14.7 regression.
3. Given a candidate window narrower than `parent`'s, the returned norm's window equals the candidate's
   own, unchanged.
4. The returned norm's `id` is deterministic given identical arguments.

## Tasks

1. Add `delegate_with_narrowing` to `src/deontic_reasoner/delegation.py`.
2. Write `tests/test_delegation_narrowing.py` covering acceptance criteria 1–4.

## Deliverables

- `project/src/deontic_reasoner/delegation.py` (modified)
- `project/tests/test_delegation_narrowing.py` (new)

## Dependencies

- `1-core-data-model` (feature) — needs `Norm`.
