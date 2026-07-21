# 3. Delegation Grant with Automatic Scope Narrowing

## What it does

The built-in rule that actually enacts a delegation: given a delegator's own norm (`parent`) and a
candidate norm describing what the delegatee is being offered (`candidate` — which may request a *wider*
scope than the delegator actually holds), derives a new norm for the delegatee whose scope is
**automatically narrowed** to the intersection of `candidate.scope` and `parent.scope` — the delegatee
can never end up with a wider scope than the delegator held, per Requirements.md → Questions: Built-in
Semantics #4, with no opt-in required.

## Inputs / outputs

```python
parent = Norm(id="p1", relation=Relation.PRIVILEGE, status=DeonticStatus.PERMITTED,
              subject="AnalyticsAgent", action="read", resource="/data/analytics/**",
              scope=Scope(contexts=frozenset({"production", "staging"})), delegable=True)
assert_norm(engine, parent)

candidate = Norm(id="c1", relation=Relation.PRIVILEGE, status=DeonticStatus.PERMITTED,
                 subject="VisualizationAgent", action="read", resource="/data/analytics/**",
                 scope=Scope(contexts=frozenset({"production", "staging", "development"})))
assert_norm(engine, candidate)

engine.facts.add(Fact("delegate", ("AnalyticsAgent", "VisualizationAgent", "c1", "p1")))
engine.run()
# A new norm now exists for VisualizationAgent whose scope.contexts == {"production", "staging"} —
# "development" was dropped, since parent didn't have it. Never a superset of parent's scope.
```

## Edge cases and failure modes

- **Scope intersection semantics** (a new `intersect_scope(a, b) -> Scope` utility):
  - `contexts`: set intersection (`a.contexts & b.contexts`).
  - `valid_from`: the later (more restrictive) of the two bounds, treating `None` as "no lower bound" —
    i.e. `None` only wins if *both* are `None`.
  - `valid_until`: the earlier (more restrictive) of the two bounds, treating `None` as "no upper
    bound" — symmetric to `valid_from`.
  - `conditions`: the union of both sides' conditions (both parent's and the candidate's constraints
    must hold — more conditions is more restrictive, consistent with narrowing), stored as a tuple
    sorted by `(predicate, args)` so the result is deterministic regardless of input order (needed for
    NFR-3 — determinism must hold even though `frozenset`/`set`-union order isn't inherently stable).
- **Non-delegable parent**: if `parent.delegable` is `False`, no grant is derived at all — delegating a
  non-delegable permission is a no-op, not an error (consistent with this reasoner's silent-non-match
  style elsewhere).
- **Missing `candidate`/`parent`**: a `delegate` fact naming a `CID`/`PID` not present in `engine.norms`
  derives nothing (dangling reference, not an error).
- **Deterministic derived id**: the granted norm's `id` must be a deterministic function of
  `(candidate.id, parent.id)` (e.g. `f"grant:{parent.id}:{candidate.id}"`), never a random/UUID id — so
  re-asserting the same `delegate` fact never produces a second, differently-`id`'d grant (relies on
  feature 2's set/dict dedup, same principle as the correlative rule's story 1 edge case).
- Registering the newly-derived norm uses a new internal `register_norm(engine, norm) -> Fact` helper
  (distinct from `assert_norm`): it sets `engine.norms[norm.id] = norm` and **returns** (does not itself
  add to `engine.facts`) `Fact("norm", (norm.id,))`, so the rule's consequent can `yield` it and let the
  engine's own fixed-point loop add it — keeping the audit trail (`2-forward-chaining-engine/3-rule-firing-audit-trail`)
  accurate about what this specific firing produced, rather than the consequent mutating `engine.facts`
  as a side effect the audit log wouldn't see.
- Widening in the *other* direction (candidate requests a **narrower** scope than parent) must pass
  through unchanged — intersection of a subset with its superset is the subset itself; this is not a
  separate code path, just the correct behavior of the same intersection.

## Acceptance criteria

1. `intersect_scope(a: Scope, b: Scope) -> Scope` implements the four field rules above.
2. `build_delegation_grant_rule()` returns a `Rule` whose antecedent is
   `("delegate", (Var("A"), Var("B"), Var("CID"), Var("PID")))`.
3. Given a `delegable=True` parent and a candidate requesting a wider `contexts` set, firing the rule
   derives a new norm for `B` whose `scope.contexts` is the intersection — a **subset** of parent's, never
   a superset (this is the direct §14.7 regression check).
4. Given a candidate requesting a narrower scope than parent, the derived norm's scope equals the
   candidate's own scope unchanged.
5. A non-delegable parent (`delegable=False`) yields no derived grant norm at all.
6. A `delegate` fact referencing a missing `CID` or `PID` yields no derived grant norm.
7. The derived grant norm's `id` is deterministic: re-asserting the identical `delegate` fact and running
   again never produces a second grant norm with a different id.

## Tasks

1. Add `intersect_scope(a, b)` to `hohfeld.py`.
2. Add `register_norm(engine, norm) -> Fact` to `hohfeld.py` (registers into `engine.norms`, returns the
   pointer `Fact` for the caller to yield — does not touch `engine.facts` itself).
3. Add `build_delegation_grant_rule()` to `hohfeld.py`, using `intersect_scope`/`register_norm`.
4. Extend `create_reasoner_engine` (`2-engine-factory-with-builtins`) to always include this rule too.
5. Write `tests/test_hohfeld_delegation_grant.py` covering acceptance criteria 1, 3–7.

## Deliverables

- `project/src/deontic_reasoner/hohfeld.py` (modified)
- `project/tests/test_hohfeld_delegation_grant.py` (new)

## Dependencies

- `1-norm-registration-and-correlative-rule`, `2-engine-factory-with-builtins`.
