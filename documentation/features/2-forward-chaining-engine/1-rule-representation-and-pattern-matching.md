# 1. Rule Representation and Pattern Matching

## What it does

Defines `Var`, `Pattern`, `Bindings`, and `Rule`, plus the core matching function that finds every
consistent set of variable bindings satisfying a rule's *full* antecedent tuple against a working-memory
fact set — a small positional join, not a general unifier (no nested terms, no occurs-check; fact/norm
arity is small and fixed).

## Inputs / outputs

- `Pattern = tuple[str, tuple[str | Var, ...]]`, e.g. `("delegate", (Var("A"), Var("B"), Var("R")))`.
- `Rule(name="delegation_triggers_oversight_duties", antecedents=(pattern,), consequent=fn, priority=0)`.
- `match_rule(rule: Rule, facts: set[Fact]) -> Iterator[Bindings]`, each `Bindings` a
  `Mapping[str, str]` from variable name to the string it was bound to.

## Edge cases and failure modes

- A variable repeated within the **same** pattern (e.g. `("likes", (Var("A"), Var("A")))`) must only
  match facts where both positions hold the same value.
- A rule with two antecedents sharing a variable must only yield bindings where **both** antecedents'
  facts agree on that variable's value — a real join, not an unconstrained cross-product of independent
  matches.
- A pattern never matches a fact of a different predicate name, or a different argument arity, even if
  some prefix of the args would otherwise line up.
- Matching against an empty fact set, or a fact set with no matches at all, yields zero bindings (not an
  error).
- Two antecedents with **no** shared variable produce the cross-product of their individual matches
  (e.g. 2 matching facts × 3 matching facts → up to 6 joint bindings), further filtered by any other
  antecedent's constraints.
- A rule with **zero** antecedents matches unconditionally exactly once, yielding a single empty
  `Bindings`. Nothing in the domain rules built so far needs this, but it's the natural degenerate case
  of "match the conjunction of N antecedents" at N=0 — not an extra code path to special-case away.

## Acceptance criteria

1. `Var` is `@dataclass(frozen=True, slots=True)` with `name: str`.
2. `Pattern = tuple[str, tuple[str | Var, ...]]` and `Bindings = Mapping[str, str]` are defined (type
   aliases, not runtime classes).
3. `Rule` is `@dataclass(frozen=True, slots=True)` with `name: str`, `antecedents: tuple[Pattern, ...]`,
   `consequent: Callable[[Bindings], Iterable[Fact]]`, `priority: int = 0`.
4. A single-antecedent, single-variable pattern matched against a fact set yields one `Bindings` per
   matching fact, correctly binding the variable to that fact's argument value.
5. A pattern with a variable repeated within itself only matches facts where every occurrence of that
   variable holds the same value.
6. A two-antecedent rule sharing a variable across antecedents yields only bindings where both
   antecedents' matched facts agree on the shared variable — verified with a case that would produce
   extra (wrong) bindings under an unconstrained cross-product.
7. A pattern never matches a fact with a different predicate name or a different argument arity.
8. Matching against an empty fact set, or one with no matching facts, yields zero bindings.
9. A rule with zero antecedents yields exactly one binding: the empty mapping.

## Tasks

1. Create `src/deontic_reasoner/rules.py`.
2. Define `Var`, `Pattern`, `Bindings`, `Rule`.
3. Implement `match_rule(rule, facts)`: match each antecedent pattern against `facts` independently,
   then join the per-antecedent candidate bindings, unifying shared variable names and rejecting
   inconsistent joins.
4. Write `tests/test_rules_matching.py` covering acceptance criteria 4–9.

## Deliverables

- `project/src/deontic_reasoner/rules.py` (new)
- `project/tests/test_rules_matching.py` (new)

## Dependencies

- `1-core-data-model` (feature) — needs `Fact`.
