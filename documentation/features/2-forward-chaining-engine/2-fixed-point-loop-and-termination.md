# 2. Fixed-Point Loop and Termination

## What it does

`ReasonerEngine` — holds working memory (`facts: set[Fact]`, `norms: dict[str, Norm]`) and a
`run(max_iterations=1000)` method that repeatedly matches every rule against current facts, fires
matched rules' consequents, collects newly-derived facts, and stops once nothing new is derived (a fixed
point) — or raises `ReasonerTimeout` if `max_iterations` is exhausted first.

## Inputs / outputs

```python
engine = ReasonerEngine(rules=[rule1, rule2])
engine.facts.add(Fact("delegate", ("A", "B", "R1")))
result = engine.run()
# engine.facts now includes every fact derivable from the initial facts.
# result.iterations -> how many iterations actually ran.
```

## Edge cases and failure modes

- A rule that re-derives a fact already in `self.facts` contributes nothing new — `set` membership
  makes this dedup automatic, and it's what keeps a monotonic rule set terminating.
- Two independent rules deriving the *same* new fact in the same iteration must not double-count or
  error — set semantics handle this for free.
- Rule firing order must be deterministic: rules are attempted `sorted(rules, key=lambda r: (-r.priority, r.name))`
  every iteration, never in raw list/dict/set iteration order, so two runs of the same input produce the
  same firing sequence.
- A pathological rule set that keeps deriving a distinct new fact every iteration (never reaching a fixed
  point) must raise `ReasonerTimeout` once `max_iterations` is hit — never hang, never silently return a
  partial result.
- Calling `run()` again after a fixed point was already reached returns immediately with no new facts —
  falls out of the loop condition naturally; worth a test, not new code.
- An empty `rules` list reaches a (trivial) fixed point immediately.

## Acceptance criteria

1. `ReasonerEngine.__init__(rules: list[Rule], closure: DeonticStatus | None = None)` stores `rules`,
   and initializes `facts: set[Fact] = set()`, `norms: dict[str, Norm] = {}`, `closure`.
2. `run(max_iterations=1000) -> RunResult` returns once an iteration produces no fact not already in
   `self.facts`.
3. A 2-hop derivation (rule A's output fact is exactly what enables rule B to fire) is fully resolved
   after `run()` — `self.facts` contains both A's and B's derived facts, not just A's.
4. A rule set engineered to always derive a fresh, never-before-seen fact raises `ReasonerTimeout` once
   `max_iterations` iterations have run, rather than returning or hanging.
5. Two fresh `ReasonerEngine` instances constructed identically and run on identical initial facts
   produce identical final `facts` sets.
6. Rules are attempted in `sorted(rules, key=lambda r: (-r.priority, r.name))` order every iteration —
   verified with two rules that could both fire in the same iteration, where firing order is observable
   (e.g. via which one's consequent runs first against a shared piece of mutable test instrumentation),
   confirming order is priority-then-name, not insertion or dict/set order.
7. An empty `rules` list makes `run()` return with `RunResult.iterations` equal to whatever the loop
   naturally produces for zero rules — pin the exact value down in the test rather than leaving it
   ambiguous.

## Tasks

1. Create `src/deontic_reasoner/engine.py`.
2. Define `RunResult` (at minimum `iterations: int`) and `ReasonerTimeout(Exception)`.
3. Implement `ReasonerEngine` with `facts`, `norms`, `closure`, and `run()`, using `match_rule` from
   `rules.py` (`2-forward-chaining-engine/1-rule-representation-and-pattern-matching`).
4. Write `tests/test_engine_fixed_point.py` covering acceptance criteria 2–7.

## Deliverables

- `project/src/deontic_reasoner/engine.py` (new)
- `project/tests/test_engine_fixed_point.py` (new)

## Dependencies

- `1-rule-representation-and-pattern-matching` (needs `Rule`, `match_rule`).
- `1-core-data-model` (feature) — needs `Fact`, `Norm`, `DeonticStatus`.
