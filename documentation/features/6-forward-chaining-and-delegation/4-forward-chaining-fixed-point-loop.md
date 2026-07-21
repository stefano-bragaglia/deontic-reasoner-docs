# 4. Forward-Chaining Fixed-Point Loop

## What it does

`forward_chain(norms, delegations=frozenset(), power_exercises=(), max_iterations=1000) -> ChainResult`
— the integration point wrapping the query engine (features 1–5): each iteration, (a) grounds every
norm currently known via `4-hohfeldian-grounding`'s `ground_norm`, (b) grounds every delegation fact via
`1-delegation-obligations-grounding`'s `ground_delegation`, (c) applies every power exercise via
`2-power-exercise-norm-generation`'s `exercise_power`, adding any genuinely new norm to the working
set — repeating until an iteration adds nothing new, or `max_iterations` is exhausted. This is what
makes requirement 14's "what new facts get added, and keep going" real: a norm created by a power's
exercise in one iteration only gets *its own* correlative grounded on the **next** iteration, since
grounding runs before power exercises are applied within each pass — genuine multi-round convergence,
not a single-pass shortcut.

## Inputs / outputs

```python
result = forward_chain(
    norms={"p1": power_norm},
    power_exercises=(("p1", {"subject": "SubAgent", "relation": Relation.PRIVILEGE,
                              "action": "read", "resource": "R2"}),),
)
# result.norms   -- contains "p1" AND the newly-derived PRIVILEGE norm
# result.rules   -- contains p1's own correlative rule (liability) AND the new norm's own (none, since
#                   PRIVILEGE has no correlative rule per 4-hohfeldian-grounding)
# result.facts   -- every norm's atom_for_norm, plus every delegation atom
# result.iterations >= 2  -- proves genuine multi-round convergence occurred
```

## Edge cases and failure modes

- **Ordering within an iteration matters**: grounding (step a) must run on the norms known *at the
  start* of that iteration, before power exercises (step c) add new ones — this is precisely what
  defers a newly-exercised norm's own grounding to the next round, making the fixed-point property
  observable and testable rather than coincidental.
- **Termination**: re-deriving an already-known norm/rule/fact (equal value, not just equal id) never
  counts as "new" — a rule set that keeps deriving genuinely new output every iteration raises
  `ForwardChainTimeout` once `max_iterations` is reached, never hangs.
- **Determinism**: identical `norms`/`delegations`/`power_exercises` inputs always produce an identical
  `ChainResult` — inherited from `ground_norm`/`ground_delegation`/`exercise_power`'s own determinism,
  this story's loop must not introduce nondeterminism (e.g. dict/set iteration order) on top.
- A `power_exercises` entry naming a `power_norm_id` absent from `norms` is a dangling reference —
  derives nothing for that entry, not an error, consistent with this reasoner's convention elsewhere.
- Calling `forward_chain` with no delegations and no power exercises still grounds every norm passed in
  (a trivial but valid case — the loop still runs at least the grounding step).

## Acceptance criteria

1. `ChainResult` has `norms: dict[str, Norm]`, `facts: frozenset[Atom]`, `rules: tuple[Rule, ...]`,
   `iterations: int`.
2. **§14.3, end-to-end**: given only a delegation fact (no norms, no power exercises),
   `forward_chain(norms={}, delegations={(delegator, delegatee, norm_id)})` produces `.rules` such that
   `is_obligatory` for the audit/liability atoms is `True` given the delegation atom.
3. **§14.6 / multi-round convergence, end-to-end**: given a `POWER` norm and one power exercise
   deriving a `RIGHT` norm (with a counterparty), `forward_chain`'s result contains the new `RIGHT`
   norm in `.norms` **and** its correlative duty rule in `.rules` — proving the second-generation norm
   was re-processed on a later iteration, not just constructed and left ungrounded. `.iterations` is
   `>= 2`.
4. A `power_exercises`/`delegations` set that never stabilizes (a synthetic pathological case
   engineered for the test) raises `ForwardChainTimeout` once `max_iterations` is reached.
5. Two calls with identical inputs produce identical `ChainResult`s (content and `iterations` count).
6. A `power_exercises` entry referencing a missing `power_norm_id` contributes nothing, without error.

## Tasks

1. Create `src/deontic_reasoner/chaining.py`.
2. Define `ChainResult`, `ForwardChainTimeout`, `forward_chain`, composing `ground_norm`,
   `ground_delegation`, `exercise_power` in the iteration order described above.
3. Write `tests/test_chaining_fixed_point.py` covering acceptance criteria 1, 4–6.
4. Write `tests/test_chaining_delegation_end_to_end.py` covering acceptance criterion 2 (§14.3).
5. Write `tests/test_chaining_power_end_to_end.py` covering acceptance criterion 3 (§14.6 +
   multi-round convergence).

## Deliverables

- `project/src/deontic_reasoner/chaining.py` (new)
- `project/tests/test_chaining_fixed_point.py` (new)
- `project/tests/test_chaining_delegation_end_to_end.py` (new)
- `project/tests/test_chaining_power_end_to_end.py` (new)

## Dependencies

- `1-delegation-obligations-grounding`, `2-power-exercise-norm-generation`,
  `3-delegation-grant-with-scope-narrowing` (this feature).
- `4-hohfeldian-grounding` (feature, story 1) — needs `ground_norm`.
