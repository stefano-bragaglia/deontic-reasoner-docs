# Requirements

Derived from `documentation/Description.md` — the minimal **Preferential (Betterness-Ordering) Dyadic
Deontic Logic** framework (Hansson dyadic `O(q|p)` + KLM preferential semantics), scoped to its 13-feature
minimal set (9 propositional core + 4 Hohfeldian agentic-extension predicates), plus the forward-chaining
layer wrapped around it. Where a requirement below corresponds to one of the numbered features in
`Description.md → Key behaviours`, that number is noted in parentheses for traceability.

## Functional Requirements

1. The system shall represent **atoms** as the base propositional vocabulary the whole system is built
   from. (feature 1)
2. The system shall represent **worlds** as `World = frozenset[str]` — the set of atoms true at that
   world (finite assignments, not a general Kripke frame). This representation is hashable and immutable
   by construction, matching the `frozen=True, slots=True` convention used elsewhere for types that must
   live in `set`s. (feature 2; see *Questions: Data Representation* #1)
3. The system shall represent **conditional rules** as `(body, head, weight)` triples — the norm
   representation. An empty/trivial body is an unconditional (default) obligation. `weight` is treated
   as coming from a trusted authoring/storage layer — the reasoning engine itself never enforces who may
   set or change it, but the engine's own consumers (whatever stores/authors rules) must ensure only the
   granting authority sets it, never the rule's own subject-agent, to prevent self-inflated priority
   under `best_worlds`'s weighted-count ordering (requirement 6). (feature 3; see *Questions: Trust and
   Governance* #2)
4. The system shall represent **hard constraints** — formulas whose violation excludes a world from
   consideration entirely, distinct from a rule violation (which merely disprefers a world). Kept in the
   requirement set as real, separate machinery (not a rule with infinite weight), but not exercised by
   any of this iteration's worked scenarios — see *Questions: Scenario Coverage* #4. (feature 4)
5. The system shall implement `violates(rule, world) -> bool`: does this world make the rule's body true
   but its head false? (feature 5)
6. The system shall implement `preferred(world_a, world_b) -> bool` under a **configurable** criterion
   over each world's violated-rule set — at minimum subset/Pareto, count, and weighted count — defaulting
   to **weighted count** when none is specified. (feature 6)
7. The system shall implement `best_worlds(antecedent) -> set[World]`: the most-preferred worlds (under
   requirement 6) among those satisfying a given antecedent formula. Enumeration shall be scoped to the
   atoms actually mentioned in the currently-loaded rule set or the antecedent formula, not the full
   power set over every atom the system has ever seen — an atom appearing in neither cannot affect which
   world is preferred and is marginalized out. (feature 7; see *Questions: Data Representation* #1 — this
   scoping approach is this vault's own extrapolation, not documented in any ingested source)
8. The system shall implement the **obligation query**: `b` is obligatory given `a` iff `b` holds at
   *every* world in `best_worlds(a)`. (feature 8)
9. The system shall implement the **permissibility query**: `b` is permitted given `a` iff `b` holds at
   *some* world in `best_worlds(a)`. (feature 9)
10. The system shall ground a Hohfeldian incident (`privilege`/`right`/`power`/`immunity`) as an
    atom-bearing fact via `norm(subject, relation, action, resource, condition)`, so the propositional
    core can reason over it. `relation=right` is what gives *directed* obligation (owed by a specific
    counterparty to a specific holder) — something the bare dyadic layer cannot express alone.
    `relation=immunity` is purely representational this iteration — groundable, forward-chainable, and
    JSON-serializable like the other three, but with no behavior yet that makes it block anything (see
    *Questions: Power and Immunity Semantics* #7). (feature 10)
11. The system shall represent the correlative bearer of a `right`/`power`/`immunity` norm via
    `counterparty(norm, agent)` — who owes the duty, is liable, or is disabled. (feature 11)
12. The system shall derive a delegation's oversight obligations (audit, revoke-on-violation, liable)
    automatically from a `delegated(delegatee, delegator, norm)` fact, via the same conditional-rule
    mechanism as requirement 3 — no separate mechanism. (feature 12)
13. The system shall evaluate `scope_matches(norm, request) -> bool` for temporal/context/condition
    applicability, so an expired or out-of-context norm does not apply to a given request. Evaluation
    resolves `(predicate_name, args)` pairs through a fixed, engine-owned predicate registry, per NFR 3
    — never `eval()`/`exec()`/`ast.literal_eval` on request data. (feature 13; see *Questions: Scope
    Evaluation* #3)
14. The system shall forward-chain a rule set to a fixed point, wrapped *around* the query engine
    (requirements 1–13), to derive: (a) new norm facts created by a power's exercise, and (b)
    delegation's derived obligations (requirement 12) — repeating until no new fact is derived, or a
    bounded iteration cap is hit. A power's exercise needs no dedicated predicate — it is an ordinary,
    domain-specific fact (e.g. `grant(...)`, `waive(...)`) used as a conditional rule's body, the same
    "no separate mechanism" principle requirement 12 already establishes for delegation, generalized to
    every other power-exercise shape (see *Questions: Power and Immunity Semantics* #6).
15. The system shall represent all core data (atoms, worlds, rules, norms, and the predicates above) as
    JSON-serializable Python dataclasses, with no custom binary format.

## Non-Functional Requirements

1. **Platform**: Python 3.14, standard library only for the baseline.
2. **Dependency ceiling**: SymPy is the sole permitted optional dependency, reserved only as a fallback
   for `scope_matches`'s condition-expression parsing if a flat predicate-registry approach proves
   insufficient — not used anywhere else, and not needed at all for the core dyadic/preferential logic
   (which requires no SAT solving).
3. **No dynamic code execution**: condition evaluation (`scope_matches` and any other condition
   evaluation) never calls `eval()`/`exec()`/`ast.literal_eval` on data that traces back to an external
   agent request — only through a fixed, engine-owned predicate registry.
4. **Determinism**: identical atoms/rules/antecedent always produce identical `best_worlds`, and
   therefore identical obligation/permissibility query answers, on every run — no dependency on
   unordered (dict/set) iteration for anything observable.
5. **Bounded termination**: the forward-chaining fixed-point loop (requirement 14) is always capped by a
   maximum iteration count; a rule set that never reaches a fixed point fails loudly rather than hanging.
6. **Tractability**: `best_worlds`/`preferred` computation must scale reasonably for realistic rule-set
   sizes — naive enumeration of the full 2^n truth-table space over all atoms is avoided by scoping
   enumeration to the atom universe of the currently-loaded rule set/antecedent (requirement 7), not the
   system's entire atom history.
7. **Test coverage**: ≥90% coverage, both aggregate and per-file, per this vault's standing project-wide
   quality gate (`CLAUDE.md → Hard Rules`).
8. **Acceptance criteria**: the nine worked scenarios from the fuller implementation spec (§14.1–14.9)
   are adopted, adapted to this framework's semantics per *Questions: Scenario Coverage* #5's detailed
   scenario-by-scenario mapping. `/features` and `/stories` should map each adapted scenario onto the
   story that implements the capability it exercises, rather than deriving new test cases from scratch.

## User Interaction Model

A Python library only — no CLI, no server, no persistence layer. A caller embeds it directly:

```python
from deontic_reasoner import (
    Atom, World, Rule, HardConstraint, PreferenceCriterion,
    Norm, Relation, best_worlds, is_obligatory, is_permitted,
    forward_chain,
)

rules = [
    Rule(body=Atom("delegated"), head=Atom("audit"), weight=10),
    Rule(body=None, head=Atom("logged"), weight=1),  # unconditional default
]
constraints = [HardConstraint(...)]

# Ground Hohfeldian facts.
norm = Norm(id="n1", relation=Relation.RIGHT, subject="FileAgent",
            action="write", resource="/out/report.csv", counterparty="filesystem")

facts = forward_chain(rules, constraints, initial_facts={...})  # fixed point, incl. derived norms

is_obligatory(Atom("audit"), given=Atom("delegated"), rules=rules, constraints=constraints,
              criterion=PreferenceCriterion.WEIGHTED_COUNT)   # -> bool
is_permitted(Atom("send_external"), given=Atom("request_received"), rules=rules, constraints=constraints)
```

Every core type round-trips through JSON (`dataclasses.asdict`/a small `from_dict`), so norms/rules can
be authored as plain JSON/dict literals as well as constructed dataclass instances directly.

## Questions: Data Representation

1. *Q: What's the exact `World`/atom representation — a `frozenset[str]` of true atoms, or a
   `Mapping[str, bool]` over a fixed, known atom universe? And how does `best_worlds` enumerate worlds
   tractably for a given antecedent, without naively generating the full 2^n truth-table space over
   every atom in the system?*
   _A: `World = frozenset[str]` (the true atoms) — hashable and immutable out of the box, matching the
   `frozen=True, slots=True` convention already used for `Fact`/`Norm` (both need to live in `set`s); a
   `Mapping[str, bool]` isn't hashable without extra wrapping and adds no expressiveness a frozenset of
   true atoms lacks. On tractable enumeration: no source documents the reference Solver's actual
   algorithm (its ingested source is README text only, silent on internal implementation/complexity) —
   this answer is this vault's own extrapolation, not a sourced claim. Scope the enumerated atom universe
   to only the atoms actually mentioned in the currently-loaded rule set or the antecedent formula, since
   an atom appearing in neither cannot affect which world is preferred and can be marginalized out —
   the same scoping principle previously used for partitioning SAT consistency checks by
   `(subject, resource)` rather than checking globally, now reapplied to atom-universe scoping instead._

## Questions: Trust and Governance

2. *Q: Is `Rule.weight` a plain non-negative number the norm-author sets directly, or does it need the
   same "only the granting authority sets it, never the norm's own subject" trust-boundary treatment
   that `Norm.priority` got in an earlier round of this project (now superseded, but the underlying
   governance concern — a delegatee inflating its own weight to outrank its delegator — is the same
   shape of problem)?*
   _A: Same treatment. `Rule.weight` decides which competing rule wins under `best_worlds`'s
   weighted-count ordering — the identical role `Norm.priority` played in the now-superseded
   `PriorityWeighted` combining algorithm — so the same governance risk applies unchanged: nothing in the
   query engine itself stops a delegatee from asserting a rule with an inflated weight to out-rank
   restrictions imposed by its delegator. Weight must be set by the rule's granting/authoring authority
   at authoring time, never mutable by the rule's own subject-agent — a requirement on whatever
   stores/authors rules, out of scope for the reasoning engine itself to enforce._

## Questions: Scope Evaluation

3. *Q: How does `scope_matches` relate to a predicate-registry-style design (a fixed, engine-owned
   mapping from predicate name to evaluator, as considered in an earlier round)? The no-`eval()`/`exec()`
   requirement on condition evaluation (NFR 3) applies regardless of which framework sits underneath, so
   this needs an answer independent of the dyadic/preferential pivot.*
   _A: Same surface. `scope_matches(norm, request)` resolves through a fixed, engine-owned registry
   mapping predicate name to a Python callable, applied to `(predicate_name, args)` pairs — never a
   string evaluated via `eval()`/`exec()`/`ast.literal_eval` — applied to a norm's temporal/context/
   condition scope instead of a bare `Condition` object. This is a trust-boundary requirement about
   externally-supplied request data, not about which deontic-logic framework computes obligation/
   permission, so it carries over unchanged from the earlier design. SymPy's expression/logic parsing
   remains the one sanctioned fallback if the flat registry ever proves insufficient for richer condition
   expressions — now even less likely to be needed, since the adopted core requires no SAT solving at
   all._

## Questions: Scenario Coverage

4. *Q: Are hard constraints (feature 4 / requirement 4) actually needed by this iteration's worked
   scenarios, or are they infrastructure to keep in reserve until a genuinely-impossible-state case
   actually shows up?*
   _A: Reserve infrastructure. None of the agentic scenarios worked out so far (scope expiry, directed
   right/duty obligations, delegation-derived contrary-to-duty obligations, a conflict resolved by
   preference weight, a broken delegation link, a power exercise, scope-narrowing on re-delegation, the
   Chisholm-paradox regression, the deontic-explosion regression) describes a genuinely *impossible*
   state — every one is naturally modeled as a violated-but-not-excluded rule, resolved by the preference
   ordering rather than by ruling a world out entirely. Hard constraints remain in the requirement set
   as specified (they're real, separate machinery in the reference Solver itself — `!formula` lines,
   distinct from weighted conditional rules) — just unexercised by the current scenario set until a
   genuinely-impossible-state case is added._
5. *Q: Which worked test scenarios ground this iteration's acceptance criteria? The previous pass at
   this project adopted the fuller implementation spec's nine worked scenarios (§14.1–14.9) wholesale,
   but several of them assumed machinery this iteration no longer builds (SAT-based
   deny-overrides conflict resolution in §14.4, immunity short-circuiting in §14.6,
   `graphlib`-based chain validation in §14.5). Do we adapt those scenarios' *intent* to the new
   weighted-count-preference semantics, write new scenarios from scratch against the 13-predicate
   minimal set, or something else?*
   _A: Adapt the same nine scenarios' intent, re-expressed under `best_worlds`/weighted-count preference:_
   - _§14.1 (scope expiry), §14.2 (directed right/duty via `norm`/`counterparty`), §14.3 (delegation
     derives its oversight obligations), §14.7 (scope-narrowing on re-delegation), and §14.8 (Chisholm's
     paradox) translate **unchanged** — none ever depended on the superseded SAT/combining-
     algorithm/`graphlib` machinery; §14.8 is now more directly on-target, since it tests exactly the
     dyadic-conditional-obligation mechanism this framework is built around._
   - _§14.4 (conflict + deny-overrides) adapts: two rules of different weight both apply;
     `is_permitted`/`is_obligatory` under weighted-count preference resolves to whichever rule's
     satisfaction costs less violated weight. The query still gets the right answer, but the explicit
     unsat-core "which norms conflict" explanation is lost along with the SAT layer — the adapted
     scenario asserts only the query outcome, not a conflict-explanation object._
   - _§14.5 (broken delegation chain) adapts: drop graph-cycle detection specifically (deferred, same as
     `graphlib`-based validation generally), but preserve the single-broken-link-denies intent — model
     each hop as a `delegated` fact plus a per-hop `valid_link` fact, and assert a downstream permission
     fails to derive when one hop's `valid_link` fact is simply absent from `initial_facts`. This falls
     out of ordinary conditional-rule matching (requirement 12's own mechanism), no dedicated
     chain-walking module needed this iteration._
   - _§14.6 (power creates a norm; immunity blocks revocation) splits: the power-creates-a-norm half
     translates directly (it's literally requirement 14a); the immunity-blocks-revocation half is out of
     scope per Q7 below — reduced to asserting an `IMMUNITY`-relation norm fact can be represented and
     forward-chained over correctly, not that it behaviorally blocks anything._
   - _§14.9 (deontic explosion containment) adapts: test that `best_worlds`/`is_obligatory` for an
     unrelated query (`AgentY`/`task2`) is unaffected by a conflicting, unrelated rule pair
     (`AgentX`/`task1`). Under this framework the containment property is **structural** (each query is
     local to its own antecedent, not a global consistency pass) rather than the result of a deliberate
     partitioning design decision the SAT version required — that structural difference is itself worth
     a regression test._

## Questions: Power and Immunity Semantics

6. *Q: How is "a power's exercise" represented as an input fact/predicate that triggers a
   norm-generating conditional rule (requirement 14a)? The minimal predicate list names `delegated`
   explicitly for delegation (predicate 12) but names no analogous predicate for a generic power
   exercise — is a new predicate needed, or does an ad hoc fact shaped like `delegated` suffice for
   every power-exercise case this iteration actually needs to handle?*
   _A: No new predicate. `delegated` earned a dedicated, named predicate because delegation is one
   single, structurally uniform pattern — but a Hohfeldian power covers a genuinely heterogeneous family
   of acts (waiving, annulling, transferring, consenting, granting, revoking), not one fixed shape, so
   there's no single analogous predicate to add. The general mechanism already suffices: a power's
   exercise is an ordinary, domain-specific fact (whatever the deployment calls it — `grant(...)`,
   `waive(...)`, etc.) as the body of an ordinary conditional rule (requirement 3), whose head asserts a
   new `norm(...)` fact — the same "no separate mechanism" principle requirement 12 already establishes
   for delegation, generalized. Whether a given deployment wants to name its own convenience predicate
   for common power-exercise shapes is a per-deployment modeling choice, not something the minimal core
   needs to anticipate._
7. *Q: Does `Relation.IMMUNITY` need any special semantic treatment in this iteration — even short of the
   full short-circuiting behavior that's explicitly out of scope per `Description.md` — or is it purely
   representational (grounded as an atom via `norm(...)`, with no behavior yet) until a later iteration
   gives it real teeth?*
   _A: Purely representational. `Relation.IMMUNITY` is groundable as a `norm(...)` fact (requirement 10)
   and carries a `counterparty` (requirement 11) like any other Hohfeldian relation, forward-chainable
   and JSON-serializable like the other three relations — but with no behavior yet that makes it
   actually block a power's exercise or a revocation. Giving it real teeth (the short-circuit-before-
   resolution behavior from the earlier, fuller design) is later-iteration work, consistent with how
   immunity short-circuiting is already listed in *Out of scope* alongside the rest of that superseded
   architecture._
