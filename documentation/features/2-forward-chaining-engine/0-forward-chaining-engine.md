# 2. Forward-Chaining Engine

## Name

`2-forward-chaining-engine`

## Summary

The reasoning core: `Rule`/`Pattern`/`Var` for antecedent pattern-matching over flat `Fact` tuples (no
general unifier — positional matching only, since norm/fact arity is small and fixed), and
`ReasonerEngine`'s fixed-point loop — match rules against working memory, fire them, collect newly
derived facts, repeat until nothing new is derived or a `max_iterations` cap raises `ReasonerTimeout`.
This exists as its own feature, before any specific domain rule (Hohfeldian correlatives, delegation
obligations) is written, because the loop's correctness properties — determinism, bounded termination,
append-only working memory — are generic engine guarantees that every rule set built on top of it
inherits for free, and are cheapest to get right in isolation.

## Requirements covered

- FR 4 — forward-chain from ground facts/norms to a fixed point.
- FR 16 (partial) — audit trail of which rule fired, from which facts, producing which derived facts (the
  engine-level half; the decision-level half of the audit trail belongs to `8-permission-resolution-pipeline`).
- NFR 3 — determinism: no reliance on dict/set iteration order; rule firing order explicitly sorted
  (priority, then name) wherever it's observable.
- NFR 4 — bounded termination: `max_iterations` always caps the loop; a non-terminating rule set fails
  loudly (`ReasonerTimeout`), never hangs.
- Requirements.md → Functional Requirements #4 (append-only working memory) — no fact is ever deleted
  from working memory by the engine itself; this is what keeps the termination argument (monotonic
  derivation) valid. (See Requirements.md → Questions: Built-in Semantics #5.)

## Dependencies

- `1-core-data-model` — `Fact`, `Norm`, and the working-memory `set`/`dict` structures the loop operates
  over must already exist.

## Stories

See `documentation/features/2-forward-chaining-engine/` for the story breakdown (added by `/stories`).
