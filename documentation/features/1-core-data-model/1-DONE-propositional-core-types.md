# 1. DONE - Propositional Core Types

## What it does

Defines the four foundational propositional-logic types, pure data with no evaluation behavior (that's
`2-preference-ordering-and-best-worlds`'s job): `Atom` (a type alias for `str`), `World` (a type alias
for `frozenset[str]` — the atoms true at that world), `Rule` (a frozen dataclass: `body`, `head`,
`weight`), and `HardConstraint` (a frozen dataclass: `forbidden`).

## Inputs / outputs

```python
Atom  # = str
World  # = frozenset[str]

Rule(body=frozenset({"delegated"}), head=frozenset({"audit"}), weight=10.0)
Rule(body=frozenset(), head=frozenset({"logged"}), weight=1.0)   # empty body = unconditional

HardConstraint(forbidden=frozenset({"sent", "blocked"}))
```

## Edge cases and failure modes

- `Rule.body` defaults to `frozenset()` (empty) — an unconditional/default rule, matching the minimal
  framework's "empty body (`⊤ → head`) is an unconditional obligation" definition. Interpreting *how*
  this behaves under `violates`/`best_worlds` is `2-preference-ordering-and-best-worlds`'s job; this
  story only needs the default to exist.
- `Rule.head` is a `frozenset[Atom]` (a conjunction), not a single `Atom` — this is a deliberate,
  minimal generalization beyond the illustrative single-atom examples in `Requirements.md`'s interaction
  model: several of the adapted worked scenarios (e.g. the broken-delegation-chain case) need a rule
  whose applicability depends on more than one atom holding simultaneously, and a conjunction is the
  simplest structure that covers both the single-atom case (a singleton frozenset) and the multi-atom
  case, without needing a full boolean-formula parser (AND is suffficient; no OR/NOT is required
  anywhere in the adapted scenarios).
- Two `Rule`s with identical `body`/`head` but different `weight` are **not** equal — `weight`
  participates in equality/hash like any other field.
- `HardConstraint.forbidden` being empty is a legal but degenerate value — this story only needs it to
  construct and hash correctly; deciding what an empty `forbidden` *means* when checking whether a world
  is excluded is deferred to `2-preference-ordering-and-best-worlds`, which is where that behavior lives.
- `Rule` and `HardConstraint` must be hashable (both are `frozen=True, slots=True` dataclasses over
  hashable field types — `frozenset`/`float`/`str`).

## Acceptance criteria

1. `Atom = str` — `Atom("x") == "x"` and behaves exactly like a string everywhere.
2. `World = frozenset[str]` — hashable, supports set operations (`&`, `|`, `<=`, etc.).
3. `Rule` is `@dataclass(frozen=True, slots=True)` with `body: frozenset[Atom] = frozenset()`,
   `head: frozenset[Atom]`, `weight: float`.
4. Two `Rule`s with identical `body`/`head` but different `weight` compare unequal.
5. `HardConstraint` is `@dataclass(frozen=True, slots=True)` with `forbidden: frozenset[Atom]`.
6. `Rule` and `HardConstraint` instances are hashable, and two instances built from identical field
   values (regardless of keyword-argument order) are equal and share a hash.

## Tasks

1. Create `src/deontic_reasoner/models.py`.
2. Define `Atom`, `World`, `Rule`, `HardConstraint`.
3. Write `tests/test_models_core.py` covering acceptance criteria 1–6.

## Deliverables

- `project/src/deontic_reasoner/models.py` (new)
- `project/tests/test_models_core.py` (new)

## Dependencies

None — first story in this feature.
