# 1. DONE - Rule Violation and Hard-Constraint Exclusion

## What it does

`violates(rule, world) -> bool` — does this world make the rule's body true but its head false? —
and `excludes(constraint, world) -> bool` — is this world one a hard constraint rules out entirely?
This is the first behavioral layer on top of `1-core-data-model`'s pure data, and everything else in
this feature builds on it.

## Inputs / outputs

```python
violates(Rule(body=frozenset({"delegated"}), head=frozenset({"audit"}), weight=10.0),
         World({"delegated"}))                       # -> True  (body true, head false)
violates(..., World({"delegated", "audit"}))          # -> False (head satisfied)

excludes(HardConstraint(forbidden=frozenset({"sent", "blocked"})), World({"sent", "blocked"}))  # -> True
excludes(..., World({"sent"}))                        # -> False (not all forbidden atoms present)
```

## Edge cases and failure modes

- An empty `body` (`frozenset()`) is a subset of every world (vacuously satisfied) — an unconditional
  rule's body is always "true," so `violates` for it reduces to just "is the head unsatisfied."
- An empty `head` (`frozenset()`) is likewise vacuously satisfied by every world — a rule with an empty
  `head` can **never** be violated, regardless of its body. Worth stating explicitly since it's easy to
  assume a rule always "does something."
- A rule whose `body` is **not** satisfied by `world` is never violated (`violates` returns `False`)
  regardless of `head` — the rule simply doesn't apply.
- `excludes` with an empty `forbidden` returns `False` for **every** world — this is the deliberate
  resolution (deferred from `1-core-data-model`) of the vacuous-truth footgun an empty `forbidden` set
  would otherwise create (treating "no atoms forbidden" as "every world violates it" would be a
  surprising, almost certainly wrong default).
- `excludes` only cares whether every atom in `forbidden` is present in `world` — extra atoms in `world`
  beyond `forbidden` don't matter.

## Acceptance criteria

1. `violates(rule, world)` returns `True` iff `rule.body <= world` and **not** `rule.head <= world`.
2. A rule with empty `body` is violated exactly when its `head` is unsatisfied by `world`.
3. A rule with empty `head` is never violated by any world, regardless of `body`.
4. A rule whose `body` is unsatisfied by `world` is never violated, regardless of `head`.
5. `excludes(constraint, world)` returns `True` iff `constraint.forbidden` is non-empty and
   `constraint.forbidden <= world`.
6. `excludes` with an empty `forbidden` returns `False` for every world tested.

## Tasks

1. Create `src/deontic_reasoner/preference.py`.
2. Implement `violates(rule, world)` and `excludes(constraint, world)`.
3. Write `tests/test_preference_violates.py` covering acceptance criteria 1–6.

## Deliverables

- `project/src/deontic_reasoner/preference.py` (new)
- `project/tests/test_preference_violates.py` (new)

## Dependencies

- `1-core-data-model` (feature, all 3 stories) — needs `Rule`, `HardConstraint`, `World`.
