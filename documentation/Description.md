# Description

## Source material

- [`references/Deontic Logic for Agent Permissions - A Formal Framework for AI Agent Governance.pdf`](references/Deontic%20Logic%20for%20Agent%20Permissions%20-%20A%20Formal%20Framework%20for%20AI%20Agent%20Governance.pdf)
  — the original motivating essay: Hohfeldian legal relations (privilege/right/power/immunity) as the
  right primitives for *multi-agent, directed* AI agent permissions, written as a critique of ad-hoc,
  path-based MCP authorization.
- [`references/query-2026-07-21-modern-deontic-framework-minimal-reasoner.md`](references/query-2026-07-21-modern-deontic-framework-minimal-reasoner.md)
  — **the primary theoretical and scope blueprint for this iteration.** Identifies the best-evidenced,
  actually-implemented "modern, commonly accepted" deontic framework — Preferential (Betterness-Ordering)
  Dyadic Deontic Logic — and specifies the *minimal* 13-predicate feature set needed for a genuinely
  working reasoner, explicitly separating that core from everything in the fuller production design that
  is robustness/explainability on top of it, not part of what makes it work. This document is what
  narrows scope for this iteration; where it disagrees with the two documents below, this one wins.
- [`references/query-2026-07-21-deontic-reasoner-implementation-spec.md`](references/query-2026-07-21-deontic-reasoner-implementation-spec.md)
  — an earlier, fuller production design (forward-chaining engine, hand-rolled SAT-based conflict
  detection with unsat-core extraction, pluggable XACML-style combining algorithms, `graphlib`-based
  delegation-chain validation). **Its conflict-detection/resolution architecture is now superseded** by
  the minimal framework's preference-ordering approach (see *What's out of scope for this iteration*,
  below) — but its dataclass/JSON data-modeling conventions and its forward-chaining fixed-point loop
  mechanics remain valid implementation patterns for the pieces of this iteration that still need them
  (norm-generating acts, delegation).
- [`references/External-Links.md`](references/External-Links.md) — tracks the
  *Defeasible-Conditional-Deontic-Logic-Solver* (Adam Labecki, under James Delgrande, SFU), now elevated
  from "precedent for a tooling choice" to **the actual reference implementation** of the framework this
  iteration adopts: it is the one real, working Python reasoner the minimal-reasoner document found that
  implements dyadic conditional rules ranked by a KLM-style preference ordering.

## What is being built (this iteration)

**The semantic reasoner only** — still no MCP server, no textual permission DSL, no governance
dashboard, no automated recourse/escalation policy. What changes from the previous pass at this
description is the *theoretical foundation and scope* of the reasoner itself: **Preferential
(Betterness-Ordering) Dyadic Deontic Logic**, built to the minimal 13-predicate feature set below, not
the fuller SAT-and-combining-algorithms design from the earlier implementation spec.

Target: Python 3.14, standard library only. This minimal framework needs no boolean-satisfiability
machinery at all for its core semantics (conflict resolution falls directly out of the preference
ordering, see below) — so the SymPy-as-fallback question from earlier onboarding rounds is now moot for
the core reasoner; if it resurfaces at all, it would only be for the agentic extension layer's
`scope_matches` condition language, exactly as before.

## Honest framing: there is no one settled "modern standard"

Deontic logic's own history is paradox → competing repair, with several repairs — dyadic detachment,
preference-based revisions, input/output logic, adaptive logics, STIT-based agency theory — still active
in parallel; nothing here claims otherwise. What earns **Preferential Dyadic Deontic Logic** the "modern,
commonly accepted" label for *this* project specifically: it resolves the largest number of classic
paradoxes with one semantic device, it's the mainstream account in the AI/nonmonotonic-reasoning
literature (not one contested position among many in pure philosophy), and — unlike the alternatives
above — it's the *only* one of the field's post-1990s repairs with a working, ingestible reference
implementation (the Solver, above). It does not natively model multi-agent directed obligations
(Hohfeldian claims/powers/immunities), doesn't have an axiomatic priority calculus as rigorous as Horty's
prioritised default logic, and is agnostic to STIT-style ought-to-do/ought-to-be distinctions. It's the
strongest implementable general-purpose core, not a complete theory of everything the field studies —
which is exactly why this project pairs it with the Hohfeldian layer from the PDF (the *agentic extension
layer*, below) rather than treating either alone as sufficient.

## Theoretical foundation

### Dyadic Deontic Logic (Hansson, 1969)

Obligation is a **primitive two-place operator** `O(q|p)` — "given p, it ought to be that q" — true at a
world *i* iff **all of *i*'s best p-worlds are q-worlds**, using a world-relative betterness ordering.
This is the device that actually resolved Chisholm's 1963 contrary-to-duty paradox, which forced deontic
logic to develop independently of normal modal logic in the first place. Unlike treating `p → O(q)` as a
material conditional (which is vacuously true whenever `p` is false and licenses bad inferences once
anything is forbidden), the dyadic operator keeps the conditional obligation meaningful independent of
whether its antecedent currently holds.

### KLM preferential semantics (Kraus, Lehmann & Magidor, 1990)

Generalizes the same device — a preference ordering over models/worlds, "if A then normally B" holds iff
B holds at the *most-preferred* A-models — from the deontic-specific case to defeasible/nonmonotonic
reasoning generally. This is what makes the framework handle **conflicting** obligations, not merely
conditional ones: the most-preferred world is the one violating the fewest (or least important)
obligations, so priority among conflicting norms falls directly out of the ordering, with no separate
conflict-resolution machinery required on top.

### The propositional core — 9 features

| # | Feature | What it does |
|---|---|---|
| 1 | **Atoms** | The propositional vocabulary (ground predicates like `sent`, `authorized`, `logged`). |
| 2 | **Worlds** | Truth assignments over the atoms — finite assignments, not a general Kripke frame. |
| 3 | **Conditional rules** `(body → head, weight)` | The norm representation: a defeasible conditional obligation. Empty body (`⊤ → head`) is unconditional. |
| 4 | **Hard constraints** `(!formula)` | Worlds violating a constraint are excluded entirely — genuinely impossible states, distinct from merely undesirable ones. |
| 5 | **Violation check** `violates(rule, world) -> bool` | Does this world make the rule's body true but its head false? |
| 6 | **Preference ordering** `preferred(world_a, world_b) -> bool` | Is *a* at least as preferred as *b*, under the chosen criterion over each world's violated-rule set? |
| 7 | **Best-worlds computation** `best_worlds(antecedent) -> set[World]` | The most-preferred worlds (under #6) among those satisfying a given antecedent — the semantic core both queries below run against. |
| 8 | **Obligation query** | `b` holds at **every** world in `best_worlds(a)`. |
| 9 | **Permissibility query** | `b` holds at **some** world in `best_worlds(a)`. |

**Preference criterion (feature #6) — the one real design freedom in the minimal set.** Three documented
options: **subset/Pareto** (violated-rule sets must be a strict subset — strictest, but leaves many
world pairs incomparable), **count** (fewer violations by raw count — total order, but treats every norm
as equally important), and **weighted count** (lower summed weight of violated rules — a high-weight norm
can outrank several low-weight ones). **This project adopts weighted count as the default**: it subsumes
count as a special case (uniform weight = 1), and it's the only one of the three that actually *resolves*
genuine normative conflicts rather than leaving them as incomparable worlds — the closest analogue to
what a production system's `priority` field needs, and it directly replaces the earlier design's
combining-algorithm machinery (deny-overrides etc.) with a single, simpler mechanism.

### The agentic extension layer — 4 predicates

The propositional core alone answers "is `b` obligatory/permissible given `a`" over a flat set of atoms —
it has no native concept of *who* is obligated to *whom*. This is exactly the gap the Hohfeldian layer
from the PDF fills:

| # | Predicate | Purpose |
|---|---|---|
| 10 | `norm(subject, relation, action, resource, condition)` | Grounds a Hohfeldian incident (`privilege`/`right`/`power`/`immunity`) as an atom-bearing fact the propositional core can reason over. `relation=right` is what gives *directed* obligation (owed by a specific counterparty to a specific holder) — something the bare dyadic `O(q\|p)` layer cannot express by itself. |
| 11 | `counterparty(norm, agent)` | The correlative bearer of a `right`/`power`/`immunity` norm — who owes the duty, who is liable, who is disabled. |
| 12 | `delegated(delegatee, delegator, norm)` | Delegation triggers the oversight duties (audit, revoke-on-violation, liable) — this is just feature #3's conditional-rule mechanism applied with the delegation event as body and the oversight duties as head; no new mechanism needed. |
| 13 | `scope_matches(norm, request)` | Temporal/context/condition applicability. Without this, every norm is unconditionally in force, which is wrong the moment any norm has an expiry. |

That's the complete minimal set: 13 predicates/functions total.

### Forward chaining — layered on top, not inside, the core

The 13-predicate core answers "what follows from the current rule set" — it does not, by itself, decide
"what new facts get added, and keep going until nothing more follows." **Powers as norm-generating acts**
(exercising a power creates a *new* norm fact) and **delegation's derived obligations** both need that
fixed-point loop wrapped around the query engine above. This is the forward-chaining requirement from
this project's original scope, and it stays in scope here — it's just layered *around* the
preferential-dyadic query engine rather than around a SAT-based one.

## What's explicitly out of scope for this iteration (superseded by the minimal framework)

Per the minimal-reasoner document's own framing, these are a robustness/explainability layer *on top of*
a working core, not part of what makes the reasoner answer queries correctly — and are dropped from this
iteration's scope, superseding the earlier (fuller) implementation spec's design:

- **SAT-based consistency checking with unsat-core extraction** — the weighted-count preference ordering
  already *is* the conflict-resolution mechanism; a separate satisfiability pre-check and
  human-readable-explanation extraction are real needs for a production system, but not for the
  reasoner to compute correct answers.
- **Pluggable XACML-style combining algorithms** (deny-overrides, permit-overrides, first-applicable,
  priority-weighted) — directly replaced by `best_worlds` under weighted-count preference.
- **`graphlib`-based delegation-chain/cycle validation** and **immunity short-circuiting** — policy-
  pipeline concerns (deciding what to do once the logic has answered a query), not semantic requirements
  of the underlying logic itself.
- **Structured audit logging** as a first-class architectural requirement.
- **Horty's prioritised default logic** — remains a cited-but-unadopted alternative conflict-resolution
  approach; not folded in.

If any of these turn out to be genuinely needed once the minimal core exists and is trustworthy, they
become later-iteration work, the same way the MCP server, DSL, and dashboard already are.

## Open items to settle in `/requirements`

- Whether `Rule.weight` (feature #3) is a plain non-negative number the norm-author sets directly, or
  needs the same "only the granting authority sets it" trust-boundary treatment `Norm.priority` got in
  the previous round's discussion.
- Exact `World`/atom representation (e.g. `frozenset[str]` of true atoms vs. `Mapping[str, bool]` over a
  fixed atom universe) and how `best_worlds` enumerates worlds in a finite, tractable way for a given
  antecedent without generating the full 2^n truth-table space naively.
- How `scope_matches` (predicate #13) relates to the earlier `Condition`/predicate-registry design (the
  no-`eval()`/`exec()` requirement from the earlier round still applies regardless of which framework is
  underneath).
- Whether hard constraints (feature #4) are needed in this iteration's actual worked scenarios, or are
  infrastructure to keep in reserve until a genuinely-impossible-state case shows up.
