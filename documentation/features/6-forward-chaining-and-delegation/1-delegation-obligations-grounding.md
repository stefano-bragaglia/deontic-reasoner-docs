# 1. Delegation Obligations Grounding

## What it does

`atom_for_delegation(delegator, delegatee, norm_id) -> Atom` and `ground_delegation(delegator,
delegatee, norm_id, weight=OBLIGATION_WEIGHT) -> tuple[Rule, Rule, Rule]` — derives the delegator's
audit duty, dyadic revoke-on-violation duty, and liability, straight from a single delegation fact, via
the **same weighted-`Rule` mechanism** `4-hohfeldian-grounding` already uses for Hohfeldian correlatives
— no separate mechanism, per `Requirements.md` FR 12. The dyadic ("revoke only if violated") condition
is represented by simply **including the violation atom in the revoke rule's body** (a conjunction),
the same technique `3-obligation-and-permissibility-queries`'s Chisholm regression already established
for conditional obligations — there is no separate "dyadic operator" type to build.

## Inputs / outputs

```python
audit_rule, revoke_rule, liable_rule = ground_delegation("Orchestrator", "DataAgent", "n1")
# audit_rule.body  == frozenset({atom_for_delegation("Orchestrator", "DataAgent", "n1")})
# audit_rule.head  == frozenset({<audit atom parameterized by "Orchestrator">})
# revoke_rule.body == frozenset({<delegation atom>, <violation atom for ("DataAgent", "n1")>})
# revoke_rule.head == frozenset({<revoke atom parameterized by "Orchestrator">})
# liable_rule.body == frozenset({<delegation atom>})
# liable_rule.head == frozenset({<liability atom relating "Orchestrator" and "DataAgent">})
```

## Edge cases and failure modes

- All three rules share the same weight (`OBLIGATION_WEIGHT`, a module-level constant, high by
  convention — the same trust-boundary note as `CORRELATIVE_WEIGHT`: whoever grounds delegation facts
  controls this weight, never the delegatee).
- `revoke_rule`'s body requires **both** the delegation atom and the violation atom — if no violation
  fact is asserted, the rule simply doesn't apply (it's not "vacuously satisfied," it just never fires),
  so `revoke` is never spuriously obligatory.
- `atom_for_delegation`/`ground_delegation` are deterministic: identical arguments always produce
  identical atoms/rules — re-deriving from the same delegation fact never produces a second,
  differently-shaped set of obligations.
- **§14.5 (broken delegation chain) is a separate, direct test — not built on `ground_delegation`.**
  Modeling a multi-hop chain is the *caller's* responsibility: construct one rule whose `body` is the
  conjunction of every hop's delegation atom *and* every hop's `valid_link` atom, with the downstream
  permission atom as its `head`. When one hop's `valid_link` atom is absent from `given`, that rule's
  body is unsatisfied, so the downstream permission is simply **not obligatory** — this is the correct
  way to read "the permission fails to derive" in this framework: `is_obligatory` for the permission
  atom goes from `True` (intact chain) to `False` (broken chain). It is *not* modeled as `is_permitted`
  going to `False`, since an atom nothing constrains is permitted by default in this framework (some
  best world can always have it true) — that default-permissive behavior is a real, deliberate property
  of this design, not a gap to work around here.

## Acceptance criteria

1. `ground_delegation(delegator, delegatee, norm_id)` returns three `Rule`s: audit, dyadic
   revoke-on-violation, and liability, each with `weight=OBLIGATION_WEIGHT` by default.
2. The audit and liability rules' `body` is exactly `{atom_for_delegation(delegator, delegatee, norm_id)}`.
3. The revoke rule's `body` is exactly `{delegation atom, violation atom}` — both required.
4. Given only the delegation atom (no violation atom) in `given`, `is_obligatory` for the revoke atom is
   `False`; given both, it is `True`.
5. `atom_for_delegation`/`ground_delegation` are deterministic across repeated calls with identical
   arguments.
6. **§14.5 regression**: constructing a two-hop chain rule (conjunction of both hops' delegation +
   `valid_link` atoms → downstream permission atom), `is_obligatory` for the permission atom is `True`
   when every hop's `valid_link` atom is present in `given`, and `False` when exactly one is missing.

## Tasks

1. Create `src/deontic_reasoner/delegation.py`.
2. Define `OBLIGATION_WEIGHT`, `atom_for_delegation`, `ground_delegation`.
3. Write `tests/test_delegation_obligations.py` covering acceptance criteria 1–5.
4. Write `tests/test_delegation_broken_chain.py` covering acceptance criterion 6 (§14.5), constructing
   the chain rule directly (no production code beyond what story 1 already provides is expected).

## Deliverables

- `project/src/deontic_reasoner/delegation.py` (new)
- `project/tests/test_delegation_obligations.py` (new)
- `project/tests/test_delegation_broken_chain.py` (new)

## Dependencies

- `1-core-data-model` (feature) — needs `Rule`, `Atom`.
- `3-obligation-and-permissibility-queries` (feature, story 1) — needs `is_obligatory`.
