# 2. Working-Memory Fact

## What it does

Defines `Fact`, the single hashable ground-proposition type used for both raw input facts and every
derived fact (including derived norms, which are pointed to as `Fact("norm", (norm.id,))`) in working
memory.

## Inputs / outputs

- `Fact("delegate", ("Orchestrator", "DataAgent", "R1"))`
- `Fact("norm", ("norm-id-1",))`
- `Fact("closed", ())` — zero-arg fact.

## Edge cases and failure modes

- Two `Fact`s with the same predicate and the same args **in the same order** must be equal and share a
  hash, so the engine's set-based deduplication (append-only working memory, per `Requirements.md` req.
  4) actually deduplicates identical derivations across iterations.
- Argument order matters: `Fact("p", ("a", "b")) != Fact("p", ("b", "a"))`.
- Zero-arg facts must construct and compare correctly.

## Acceptance criteria

1. `Fact` is `@dataclass(frozen=True, slots=True)` with `predicate: str` and `args: tuple[str, ...]`.
2. `Fact("p", ("a", "b")) == Fact("p", ("a", "b"))` and both share a hash.
3. `Fact("p", ("a", "b")) != Fact("p", ("b", "a"))`.
4. `Fact` instances placed into a `set[Fact]` deduplicate correctly (adding the same fact twice leaves
   the set with one element).

## Tasks

1. Add `Fact` to `src/deontic_reasoner/models.py`.
2. Write `tests/test_models_fact.py` covering acceptance criteria 1–4.

## Deliverables

- `project/src/deontic_reasoner/models.py` (modified)
- `project/tests/test_models_fact.py` (new)

## Dependencies

None — independent of `1-core-enums-and-identifiers`; ordered second by convention only.
