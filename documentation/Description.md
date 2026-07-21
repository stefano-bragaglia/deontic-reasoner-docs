# Description

This synthesizes three sources: the original motivating essay
[`references/Deontic Logic for Agent Permissions - A Formal Framework for AI Agent Governance.pdf`](references/Deontic%20Logic%20for%20Agent%20Permissions%20-%20A%20Formal%20Framework%20for%20AI%20Agent%20Governance.pdf)
(the general intuition — Hohfeldian legal relations as the right primitives for multi-agent, directed AI
agent permissions); the minimal-scope blueprint
[`references/query-2026-07-21-modern-deontic-framework-minimal-reasoner.md`](references/query-2026-07-21-modern-deontic-framework-minimal-reasoner.md)
(**primary** — defines the actual theoretical framework and the minimal feature set for this iteration);
and the fuller
[`references/query-2026-07-21-deontic-reasoner-implementation-spec.md`](references/query-2026-07-21-deontic-reasoner-implementation-spec.md)
(used only where the minimal document is silent — data representation and forward-chaining mechanics —
since its own conflict-detection/resolution architecture is superseded by the minimal document's approach;
see *Out of scope* below). The real, working
[Defeasible-Conditional-Deontic-Logic-Solver](references/External-Links.md) is the reference
implementation for the framework this project adopts.

## Purpose

A Python library implementing a **semantic reasoner for AI agent permissions**, grounded in a real,
implementable deontic-logic framework rather than ad-hoc, path-based access control. It answers
obligation and permission questions about agents, actions, and resources, and models the
Hohfeldian relations (privilege, right, power, immunity) needed to express *directed*, multi-agent
obligations (who owes what to whom) — not just a flat, ungrounded "is this allowed" boolean.

**This iteration is the reasoner only.** Integrating it behind an MCP server (tool/resource
authorization for AI agents) is explicitly deferred to a separate, later iteration — nothing here builds
a server, a wire protocol, or MCP-specific tooling.

## Inputs and outputs

**Inputs** (all as Python data, no external file format assumed yet):
- **Atoms** — ground propositional facts (e.g. `sent`, `authorized`, `logged`).
- **Conditional rules** — `(body → head, weight)`, the norm representation; an empty body is an
  unconditional/default obligation.
- **Hard constraints** — formulas whose violation excludes a world entirely (genuinely impossible
  states, not merely disfavored ones).
- **Hohfeldian norm facts** — `norm(subject, relation, action, resource, condition)` grounding a
  privilege/right/power/immunity incident, plus `counterparty(norm, agent)` for the correlative bearer.
- **Delegation facts** — `delegated(delegatee, delegator, norm)`.
- **Scope/context facts** — whatever `scope_matches(norm, request)` needs to decide temporal/context/
  condition applicability for a concrete request.

**Outputs**:
- **Obligation query answers** — is `b` obligatory given `a`: true iff `b` holds at *every* world in
  `best_worlds(a)`.
- **Permissibility query answers** — is `b` permitted given `a`: true iff `b` holds at *some* world in
  `best_worlds(a)`.
- **Forward-chained derived facts** — new norm facts created by a power's exercise, and delegation's
  derived oversight obligations (audit, revoke-on-violation, liable), produced by iterating the rule set
  to a fixed point.

No CLI, no server, no persistence layer this iteration — a caller embeds the library directly and gets
back query answers / derived facts as plain Python objects.

## Key behaviours

The core framework is **Preferential (Betterness-Ordering) Dyadic Deontic Logic**: Hansson's dyadic
obligation operator `O(q|p)` (1969) — true at world *i* iff *all* of *i*'s best `p`-worlds are
`q`-worlds — combined with KLM preferential semantics (Kraus, Lehmann & Magidor, 1990), which generalizes
the same "most-preferred model" device to handle **conflicting** obligations: the most-preferred world is
the one violating the fewest/least-important norms, so conflict resolution falls directly out of the
ordering, no separate machinery required. The dyadic operator specifically is what avoids treating
`p → O(q)` as a material conditional — the classic failure mode (vacuously true when `p` is false,
licensing bad inferences once anything is forbidden) that caused Chisholm's 1963 contrary-to-duty paradox.

**Minimal propositional core (9 features)**, mirroring the real Solver's own interface:

| # | Feature | What it does |
|---|---|---|
| 1 | Atoms | The propositional vocabulary the system is built from. |
| 2 | Worlds | Truth assignments over the atoms — finite assignments, not a general Kripke frame. |
| 3 | Conditional rules `(body → head, weight)` | Defeasible conditional obligations; empty body = unconditional. |
| 4 | Hard constraints `(!formula)` | Exclude violating worlds entirely — impossible, not just undesirable. |
| 5 | `violates(rule, world) -> bool` | Does this world make the rule's body true but head false? |
| 6 | `preferred(world_a, world_b) -> bool` | Is *a* at least as preferred as *b*, under the chosen criterion? |
| 7 | `best_worlds(antecedent) -> set[World]` | Most-preferred worlds (under #6) satisfying the antecedent. |
| 8 | Obligation query | `b` holds at every world in `best_worlds(a)`. |
| 9 | Permissibility query | `b` holds at some world in `best_worlds(a)`. |

Feature #6 needs one of three preference criteria — **weighted count** (lower summed weight of violated
rules wins) is adopted as the default: it subsumes plain **count** (uniform weight = 1) as a special
case, and unlike **subset/Pareto** (strict-subset comparison, leaves many world pairs incomparable), it's
the only one that actually *resolves* genuine conflicts rather than leaving them undecided.

**Agentic extension layer (4 predicates)**, bridging the propositional core to multi-agent, directed
obligations (the gap plain `O`/dyadic-`O(q|p)` can't express on its own):

| # | Predicate | Purpose |
|---|---|---|
| 10 | `norm(subject, relation, action, resource, condition)` | Grounds a Hohfeldian incident as an atom the core can reason over. `relation=right` gives directed obligation (owed by a specific counterparty to a specific holder). |
| 11 | `counterparty(norm, agent)` | Who owes the duty / is liable / is disabled, for a `right`/`power`/`immunity` norm. |
| 12 | `delegated(delegatee, delegator, norm)` | Triggers oversight duties (audit, revoke-on-violation, liable) — just feature #3's conditional-rule mechanism, no new machinery. |
| 13 | `scope_matches(norm, request)` | Temporal/context/condition applicability, so an expired or out-of-context norm doesn't apply. |

That's the complete minimal set: **13 predicates/functions total**.

**Forward chaining stays in scope** (this was part of the original ask) but is layered *around* this
query engine rather than inside it: the 13-predicate core answers "what follows from the current rule
set," while a fixed-point loop wrapped around it handles "what new facts get added" — specifically,
**powers as norm-generating acts** (exercising a power creates a new norm fact) and **delegation's
derived obligations**. Data representation follows the implementation spec's conventions where the
minimal document is silent on them: plain, JSON-serializable Python dataclasses (no custom binary
format), since nothing about the choice of query engine underneath changes how facts/norms should be
represented.

## Out of scope

- **The MCP server integration** — explicitly deferred to its own later iteration, per this round's
  instruction; nothing here concerns tool/resource authorization wiring, transport, or MCP-specific
  schemas.
- A textual permission DSL/parser, a governance dashboard, automated recourse/escalation policy — all
  later-iteration work, same as before.
- **Superseded by the minimal framework's own preference-ordering mechanism** (previously specified in
  the fuller implementation spec, now dropped since they're a robustness/explainability layer on top of
  a working core, not part of what makes it answer queries correctly):
  - SAT-based consistency checking with unsat-core extraction.
  - Pluggable XACML-style combining algorithms (deny-overrides, permit-overrides, first-applicable,
    priority-weighted) — replaced by `best_worlds` under weighted-count preference.
  - `graphlib`-based delegation-chain/cycle validation and immunity short-circuiting — policy-pipeline
    concerns, not semantic requirements of the underlying logic.
  - Structured audit logging as a first-class architectural requirement.
  - Horty's prioritised default logic — a cited-but-unadopted alternative to weighted-count preference.
- No external dependency beyond the standard library. SymPy remains the sole pre-approved fallback, and
  only plausibly for `scope_matches`'s condition-expression language if the flat predicate-registry
  approach proves insufficient — the core dyadic/preferential logic needs no SAT solving at all, so this
  fallback is far less likely to be needed than it was under the earlier (superseded) design.

## Open questions

1. Is `Rule.weight` (feature #3) a plain value the norm-author sets directly, or does it need the same
   "only the granting authority sets it" trust-boundary treatment considered for `Norm.priority` in an
   earlier round?
2. What's the exact `World`/atom representation (e.g. `frozenset[str]` of true atoms vs.
   `Mapping[str, bool]` over a fixed atom universe), and how does `best_worlds` enumerate worlds
   tractably for a given antecedent without naively generating the full 2^n truth-table space?
3. How does `scope_matches` relate to a predicate-registry-style design (the no-`eval()`/`exec()`
   requirement on condition evaluation still applies regardless of which framework sits underneath)?
4. Are hard constraints (feature #4) actually needed by this iteration's worked scenarios, or are they
   infrastructure to keep in reserve until a genuinely-impossible-state case shows up?

## Tech stack

- **Project**: `deontic-reasoner`, Python 3.14, standard library only (SymPy as the sole possible
  fallback dependency, per *Out of scope* above).
- Library only this iteration — no CLI, no server, no database.
- Repos already provisioned (from `/setup`, unaffected by this description revision): docs
  (`deontic-reasoner-docs`, public) and code (`deontic-reasoner`, public), both on GitHub.
