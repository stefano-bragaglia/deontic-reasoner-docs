# 2. Engine Factory with Built-ins Always Included

## What it does

`create_reasoner_engine(extra_rules=(), closure=None) -> ReasonerEngine` — the entry point real callers
use, guaranteeing the correlative rule (and, once later stories in this feature add them, the delegation
rules) is always present, regardless of what the caller passes. This is how Requirements.md → Questions:
Built-in Semantics #3 ("automatic, built-in, no per-domain opt-in required") is actually delivered: since
feature 2's `ReasonerEngine(rules=[...])` constructor takes a plain caller-supplied list and has no
knowledge of Hohfeldian semantics (feature 2 doesn't depend on feature 3), the guarantee has to live in a
factory function here, not in the raw constructor.

## Inputs / outputs

```python
engine = create_reasoner_engine()
# engine.rules already includes the correlative rule, even though nothing was passed.

engine2 = create_reasoner_engine(extra_rules=(my_domain_rule,))
# engine2.rules includes the correlative rule AND my_domain_rule.
```

## Edge cases and failure modes

- The correlative rule's consequent closes over the *specific* engine instance it's built for (it looks
  up `engine.norms`) — constructing two engines via this factory must not have one's correlative rule
  accidentally bound to the other's `norms` dict. Verified by running two engines independently and
  confirming each only sees its own derived facts.
- Calling `ReasonerEngine(rules=[...])` directly (feature 2's raw constructor) still works and remains
  useful for testing feature 2 in isolation — but it does **not** get the correlative-rule guarantee.
  This factory is the one real callers (and later, `8-permission-resolution-pipeline`) are expected to
  use; that distinction is called out explicitly so later features don't accidentally reach for the raw
  constructor.
- `extra_rules=()` (the default — no extras) still gets the built-in correlative rule; "no extras" must
  not be confused with "no rules at all."

## Acceptance criteria

1. `create_reasoner_engine()` returns a `ReasonerEngine` whose `.rules` contains exactly one rule (the
   correlative rule from `1-norm-registration-and-correlative-rule`).
2. `create_reasoner_engine(extra_rules=(my_rule,))` returns an engine whose `.rules` contains both the
   correlative rule and `my_rule`.
3. Asserting a `RIGHT` norm via `assert_norm` on an engine built by this factory and calling `.run()`
   derives the `duty` fact without the caller having passed the correlative rule explicitly.
4. Two engines constructed independently via this factory never see each other's derived facts (each
   engine's correlative rule is correctly bound to its own `norms`/`facts`).

## Tasks

1. Add `create_reasoner_engine(extra_rules=(), closure=None)` to `hohfeld.py`: construct a
   `ReasonerEngine` with an empty rule list, then set `engine.rules = [build_correlative_rule(engine), *extra_rules]`
   (the rule needs the engine instance to exist first, since its consequent closes over it).
2. Write `tests/test_hohfeld_factory.py` covering acceptance criteria 1–4.

## Deliverables

- `project/src/deontic_reasoner/hohfeld.py` (modified)
- `project/tests/test_hohfeld_factory.py` (new)

## Dependencies

- `1-norm-registration-and-correlative-rule`.
