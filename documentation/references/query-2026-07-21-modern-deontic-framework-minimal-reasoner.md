---
query: >
  Assuming to implement the most modern and commonly accepted framework for
  deontic logic, please name it and provide the minimal set of features and
  predicates to obtain a fully working implementation of the reasoner.
date: 2026-07-21
answered_from: wiki/ only (see citations throughout)
---

# The Most Modern, Commonly-Accepted Deontic Framework — and a Minimal Implementation

## Honest framing first

Deontic logic does not have a single unanimous "winner" the way, say, epistemic logic has S5 for common knowledge. The field's own history (see `raw/query-2026-07-21-deontic-logic-document.md` for the full trajectory) is a sequence of paradox → competing repair, with several repairs still actively defended in parallel: the SEP survey the vault ingested treats dyadic detachment, preference-based revisions of Inheritance, input/output logic, adaptive logics, and STIT-based agency theory as **live, co-existing** research programs, not a settled succession. [[Deontic Logic (Stanford Encyclopedia of Philosophy)]] Anyone claiming deontic logic has one settled "modern standard" is oversimplifying.

That said, one specific combination is the best-evidenced candidate in this vault for "modern and commonly accepted" **in the sense that matters for building a working reasoner**: it resolves the largest number of classic paradoxes with a single semantic device, it is explicitly the mainstream account in the *AI/nonmonotonic-reasoning* literature (as opposed to being one contested position among many in pure philosophy), and — unlike input/output logic, adaptive logics, or STIT/agency theory, all of which the wiki documents but for which no runnable implementation was found — it is the *only* one of the field's post-1990s repairs the wiki has actually seen implemented as working code.

## Naming it: Preferential (Betterness-Ordering) Dyadic Deontic Logic

This is a synthesis name for a two-part stack the wiki's sources converge on independently:

1. **Dyadic Deontic Logic** (Bengt Hansson, 1969) — obligation is a primitive two-place operator `O(q|p)` ("given p, it ought to be that q"), true at world *i* iff *all the i-best p-worlds are q-worlds*, using a world-relative **betterness ordering** `≥i`. This is the response that actually resolved Chisholm's 1963 contrary-to-duty paradox — the paradox that, per the field's own history, is "widely regarded as the event that forced deontic logic to develop independently of normal modal logic." [[Dyadic Deontic Logic]], [[Contrary-to-Duty Obligations and Chisholm's Paradox]]

2. **KLM preferential semantics** (Kraus, Lehmann & Magidor, 1990) — generalizes exactly the same device (a preference/typicality ordering over models, "if A then normally B" holds iff B holds at the most-preferred A-models) from the deontic-specific case to defeasible/nonmonotonic reasoning generally. This is the standard model-theoretic account of defeasible conditionals in mainstream nonmonotonic AI — not a niche philosophical position — and it is what makes the framework handle **conflicting** obligations (§1.8 of the history document) rather than merely conditional ones: the most-preferred world is the one violating the fewest (or least important) obligations, so priority among conflicting norms falls directly out of the ordering, with no separate machinery needed. [[KLM Preferential Semantics and the Defeasible-Conditional-Deontic-Logic Solver]]

The wiki's evidence that this combination, and not some other repair, is the one actually used in practice: a real, MIT-licensed, working Python tool — the *Defeasible-Conditional-Deontic-Logic-Solver* (Adam Labecki, under James Delgrande, Simon Fraser University) — implements precisely this stack: dyadic conditional rules, ranked by a KLM-style preference ordering, answering both obligation and permissibility queries against the most-preferred worlds. It is the only end-to-end runnable deontic reasoner this vault has studied. [[KLM Preferential Semantics and the Defeasible-Conditional-Deontic-Logic Solver]]

**What this framework is not**, for calibration: it does not natively model multi-agent directed obligations (Hohfeldian claims/powers/immunities — that's a separate layer, §3 below), it does not have an axiomatic priority calculus as rigorous as Horty's prioritised default logic (cited in the wiki but never ingested from a primary source), and it is agnostic to agency/STIT-style "ought-to-do vs. ought-to-be" distinctions (John Horty's separate contribution). It is the strongest **general-purpose, implementable core**, not a complete theory of everything the field studies. [[Deontic Conflicts and Defeasible Deontic Logic]], [[John Horty]]

## Minimal feature set — the propositional core

This is deliberately smaller than the full production design in `raw/query-2026-07-21-deontic-reasoner-implementation-spec.md`: the smallest set of features that makes Preferential Dyadic Deontic Logic actually run, mirroring the real solver's documented interface. [[KLM Preferential Semantics and the Defeasible-Conditional-Deontic-Logic Solver]]

| # | Feature | What it does |
|---|---|---|
| 1 | **Atoms** | The propositional vocabulary the whole system is built from (ground predicates like `sent`, `authorized`, `logged`). |
| 2 | **Worlds** | Truth assignments over the atoms — the only "possible worlds" this framework needs are these finite assignments, not a general Kripke frame. |
| 3 | **Conditional rules** `(body → head, weight)` | The norm representation: a defeasible conditional obligation. An empty body (`⊤ → head`) is an unconditional/default obligation. |
| 4 | **Hard constraints** `(!formula)` | Worlds violating a constraint are excluded from consideration *entirely*, not merely dispreferred — the mechanism for genuinely impossible states, as distinct from merely undesirable ones. |
| 5 | **Violation check**: `violates(rule, world) -> bool` | Does a given world make this rule's body true but its head false? |
| 6 | **Preference ordering**: `preferred(world_a, world_b) -> bool` | World *a* is at least as preferred as *b* iff *a*'s violated-rule set relates to *b*'s under the chosen criterion. |
| 7 | **Best-worlds computation**: `best_worlds(antecedent) -> set[World]` | The most-preferred worlds (under #6) among those satisfying a given antecedent formula — this is the semantic core both queries below are answered against. |
| 8 | **Obligation query**: `R, a ⊨ b` (obligatory) | *b* holds at **every** world in `best_worlds(a)`. |
| 9 | **Permissibility query**: `R, a ⊨ b` (permissible) | *b* holds at **some** world in `best_worlds(a)`. |

### The one genuine design decision inside this minimal set: which preference criterion

Feature #6 needs exactly one of three criteria — this is the only place the minimal implementation has real freedom, and it's directly documented from the working solver, not invented:

- **Subset (Pareto)**: world *a* ⪰ world *b* iff the rules *a* violates are a *subset* of the rules *b* violates. Strictest, most philosophically conservative (never trades one violation for "fewer, but different" violations), but yields the fewest total-ordered comparisons — many world pairs are simply incomparable.
- **Count**: *a* ⪰ *b* iff *a* violates no more rules, by raw count, than *b* does. Total order, simple, but treats all norms as equally important.
- **Weighted count**: *a* ⪰ *b* iff *a*'s violated rules sum to no more weight than *b*'s. This is the one that gives conflict *resolution*, not just conflict *detection* — a high-weight norm can outrank several low-weight ones — and is the closest analogue to what a production system's `priority` field would need. [[KLM Preferential Semantics and the Defeasible-Conditional-Deontic-Logic Solver]], [[Deontic Conflicts and Defeasible Deontic Logic]]

For a "fully working" reasoner with the least additional design work, **weighted count** is the recommended default: it subsumes count (uniform weight = 1) as a special case, and it's the only one of the three that actually resolves genuine normative conflicts (Plato's Dilemma-style cases, §1.8 of the history document) rather than merely leaving them as incomparable worlds.

## Minimal extension layer — making it usable for agents

The propositional core above answers "is *b* obligatory/permissible given *a*" over a flat set of atoms — it has no native concept of *who* is obligated to *whom*, which is exactly the gap the Hohfeldian layer fills, and exactly why the agentic-use-case document layered it on top of a dyadic/SDL core rather than treating it as sufficient by itself. The minimal predicates to bridge from the propositional engine above to a genuinely multi-agent reasoner:

| # | Predicate | Purpose |
|---|---|---|
| 10 | `norm(subject, relation, action, resource, condition)` | Ground a Hohfeldian incident (`privilege`/`right`/`power`/`immunity`, §3.2 of the implementation spec) as an atom-bearing fact the propositional core can reason over — `relation=right` is what gives *directed* obligation (owed *by* a specific counterparty *to* a specific holder), which the bare `O`/dyadic-`O(q\|p)` layer cannot express on its own. [[Hohfeldian Incidents]] |
| 11 | `counterparty(norm, agent)` | The correlative bearer of a `right`/`power`/`immunity` norm — who owes the duty, who is liable, who is disabled. |
| 12 | `delegated(delegatee, delegator, norm)` | The one derivation rule every agentic deployment needs: delegation triggers contrary-to-duty obligations (`audit`, `revoke-on-violation`, `liable`) — this is just conditional-rule feature #3 applied with the delegation event as body and the oversight duties as head, so no new mechanism is required beyond what's already in the core. [[AI Agent Permission Calculus]] |
| 13 | `scope_matches(norm, request)` | Temporal/context/condition applicability check — without this, every norm is treated as unconditionally in force, which is wrong the moment any norm has an expiry. |

That's it: 13 predicates/functions total (9 for the propositional preferential-dyadic core, 4 to make it agent-aware) is the minimal complete set. Everything in the fuller production spec — SAT-based consistency checking with unsat-core extraction, `graphlib`-based delegation-cycle detection, pluggable XACML-style combining algorithms, audit logging, scoped consistency-check partitioning — is a **robustness/explainability layer on top of this minimal core**, not part of what makes the reasoner *work* in the first place. The weighted-count preference ordering (feature #6) already *is* the conflict-resolution mechanism; a SAT consistency check is only needed if you additionally want a yes/no "is this norm set even consistent" answer *before* asking which world wins, and unsat-core extraction is only needed if you want a human-readable explanation of *why* — both real requirements for a production system (per the implementation spec's FR-4/FR-6), neither strictly required for the reasoner to compute correct obligation/permission answers. [[SAT Solving and DPLL]]

## What's deliberately left out of "minimal"

- **Powers as norm-generating acts** (a power's exercise creates a *new* norm fact) needs a forward-chaining fixed-point loop around the query engine above, not just the query engine itself — the 13-predicate core answers "what follows from the current rule set," a full reasoner also needs "what new facts get added, and keep doing that to a fixed point." This is the forward-chaining requirement from the original request, layered on top of, not inside, the preferential-dyadic core. [[Experta - Forward-Chaining Rule Engine]]
- **Immunity short-circuiting** and **liability-chain validation** are policy-pipeline concerns (§8.2, §9 of the implementation spec), not semantic requirements of the underlying logic — they decide *what to do* once the logic has answered a query, not part of answering it.
- **Horty's prioritised default logic** is not part of this framework and is not folded in here — it remains a cited-but-unstudied alternative approach to the same conflict-resolution problem weighted-count preference is solving; the wiki flags this as its most significant standing theoretical gap. [[John Horty]], [[Deontic Conflicts and Defeasible Deontic Logic]]

## Sources

[[Deontic Logic (Stanford Encyclopedia of Philosophy)]] · [[Dyadic Deontic Logic]] · [[Contrary-to-Duty Obligations and Chisholm's Paradox]] · [[KLM Preferential Semantics and the Defeasible-Conditional-Deontic-Logic Solver]] · [[Deontic Conflicts and Defeasible Deontic Logic]] · [[Hohfeldian Incidents]] · [[AI Agent Permission Calculus]] · [[Standard Deontic Logic]] · [[John Horty]] · [[Experta - Forward-Chaining Rule Engine]] · [[SAT Solving and DPLL]]
