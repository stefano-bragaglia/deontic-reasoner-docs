# 3. Obligation and Permissibility Queries

## Name

`3-obligation-and-permissibility-queries`

## Summary

The actual "answer a permission question" surface of the reasoner, built directly on
`2-preference-ordering-and-best-worlds`'s `best_worlds`: the obligation query (`b` holds at *every*
world in `best_worlds(a)`) and the permissibility query (`b` holds at *some* world in `best_worlds(a)`).
This is the dyadic operator `O(q|p)` and its permission dual made concrete and callable.

## Requirements covered

- FR 8 — obligation query.
- FR 9 — permissibility query.
- Acceptance criteria (Requirements.md → Questions: Scenario Coverage #5's adaptation):
  - §14.4 — a conflict between two differently-weighted rules resolves to whichever rule's satisfaction
    costs less violated weight; the query returns the right answer without a separate conflict-
    explanation object (that explanation capability was dropped along with the superseded SAT layer).
  - §14.8 — the Chisholm's-paradox regression: two dyadic conditional obligations with complementary
    conditions never both fire, and the query engine derives the one matching the actual facts without
    contradiction.
  - §14.9 — deontic-explosion containment: an unrelated query's answer is unaffected by a conflicting,
    unrelated rule pair, because query evaluation is inherently local to its own antecedent — a
    structural property of `best_worlds`, not a deliberate partitioning design decision the way the
    superseded SAT layer needed.

## Dependencies

- `2-preference-ordering-and-best-worlds`.

## Stories

See `documentation/features/3-obligation-and-permissibility-queries/` for the story breakdown (added by
`/stories`).
