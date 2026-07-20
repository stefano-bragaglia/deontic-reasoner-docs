# Description

## Source material

- [`references/Deontic Logic for Agent Permissions - A Formal Framework for AI Agent Governance.pdf`](references/Deontic%20Logic%20for%20Agent%20Permissions%20-%20A%20Formal%20Framework%20for%20AI%20Agent%20Governance.pdf)
  — the original motivating essay: Hohfeldian + deontic-logic foundation for AI agent authorization,
  written as a critique of ad-hoc, path-based MCP authorization. Informal in places (states an
  algorithm and a conflict-resolution rule without formal definition).
- [`references/query-2026-07-21-deontic-reasoner-implementation-spec.md`](references/query-2026-07-21-deontic-reasoner-implementation-spec.md)
  — **the primary architectural blueprint for this iteration.** A formal design spec that takes the
  PDF's model as its target problem and resolves every place the PDF was informal (conflict resolution,
  closure, contrary-to-duty obligations, the stdlib-vs-SymPy question) into a concrete, implementable
  design: data model, forward-chaining engine, hand-rolled SAT-based conflict detection, pluggable
  combining algorithms, delegation-chain validation, draft requirements (FR/NFR), a risk table, and nine
  worked test scenarios. Where this document and the PDF disagree on specifics, this one wins — it was
  written specifically to resolve the PDF's ambiguities under this project's actual constraints (Python
  3.14, stdlib-first, SymPy-as-fallback, dataclasses/JSON, forward chaining).
- [`references/External-Links.md`](references/External-Links.md) — external precedent (the SymPy-based
  deontic solver referenced when discussing tooling choice).

## What is being built (this iteration)

**The semantic reasoner only** — the core logical engine that decides permission questions. Explicitly
**not** in scope for this iteration: the MCP server, a textual permission DSL/parser (rules are authored
directly as Python `Rule` objects — the implementation spec explicitly leaves a declarative surface
syntax as a later, optional addition, §15), the governance dashboard, and automated recourse/escalation
policy. These remain later-iteration work, to be picked up once the reasoner itself exists and is
trustworthy.

Target: Python 3.14, standard library only. SymPy (`sympy.logic`) is the one pre-approved fallback
dependency, reserved for exactly one escape hatch — see *Resolved: stdlib vs. SymPy* below.

## Theoretical foundation

### Deontic operators, and a deliberate departure from Standard Deontic Logic

`O` (obligatory) is primitive; permission and prohibition are defined from it: `P(p) ≡ ¬O(¬p)`,
`F(p) ≡ O(¬p) ≡ ¬P(p)`. Unlike textbook Standard Deontic Logic (SDL), this reasoner does **not** globally
enforce SDL's "obligations cannot conflict" axiom — two delegators can independently impose genuinely
incompatible duties, and a logic that rules this out by fiat can't represent that situation, only crash
on it. Conflicts are instead **detected** (via a consistency check) and **resolved** by an explicit,
configurable policy — see *Architecture* below.

### Hohfeldian relations — the multi-agent layer

Plain `O`/`P`/`F` is agent-neutral: it can't say *who* a duty is owed to, or *who* can change *whose*
normative position. Every norm is one of four Hohfeldian incidents:

| Relation | Holder can... | Correlative (counterparty has) | Opposite (holder lacks) |
|---|---|---|---|
| **Privilege** | do X, no duty not to | no-right | duty |
| **Right** (claim) | demand X of someone | duty (on someone) | no-claim |
| **Power** | change a normative position (grant/revoke) | liability | disability |
| **Immunity** | be secure against a position-change | disability (in someone) | liability |

`Privilege` corresponds to plain `P`; `Right` corresponds to a *directed* `O` (owed by a specific
counterparty, to a specific holder) — directedness plain SDL cannot express. `Power` and `Immunity` have
no SDL analogue at all: they concern **norm-changing acts**, and are modeled as ordinary derivation rules
whose consequent is itself a new norm (a power exercised is a rule firing that asserts a fact), not as a
deontic operator over propositions.

### Conditional (dyadic) obligation — why delegation needs it

A norm that only fires given some other condition is written `O(φ | ψ)` as a **primitive dyadic
operator** — not `ψ → O(φ)`, which is known to misbehave (the classic "ashtray"/"gentle murder"
problems). This matters directly for delegation: "if B violates the terms, A ought to revoke B's access"
is a **contrary-to-duty obligation**, structurally identical to Chisholm's 1963 paradox. Representing it
dyadically (`O(revoke | violation)`) rather than as a material conditional avoids the exact trap SDL fell
into — the obligation to revoke exists and is reasonable about *before* any violation, without becoming
vacuously true or licensing bad inferences once anything is forbidden.

### Obligation chains from delegation

When agent A delegates a permission to agent B, the reasoner derives the correlative obligations
automatically:

```
delegate(A, B, permission(read, R, S)) →
    O(A, audit(B.actions(R))) ∧
    O(A, revoke(B, R) | violation(B, R)) ∧      ← dyadic, per above
    liable(A, damages(B.actions(R)))
```

### The closure problem

The normative status of an action nobody explicitly addressed is a **per-domain configuration value**,
not a hardcoded answer (SDL's own "Exhaustion" theorem forces a closed-world answer real normative
systems don't actually have):

- **Permissive closure**: unaddressed ⇒ permitted (a general-purpose assistant with explicit
  prohibitions).
- **Prohibitive closure**: unaddressed ⇒ forbidden (a high-stakes agent, e.g. one that moves money, with
  explicit permissions).

## Architecture (resolved by the implementation spec)

- **Data model**: `frozen=True, slots=True` dataclasses — `Agent`, `Resource`, `Norm`, `Fact`, `Scope`,
  `Condition`, `Provenance`, plus `Relation` and `DeonticStatus` enums. All JSON-round-trippable via
  `dataclasses.asdict`/a small `from_dict`, no custom binary format.
- **Forward-chaining engine**: a fixed-point loop — match rules against working-memory facts, fire,
  collect new facts, repeat until nothing new is derived (or a `max_iterations` cap raises
  `ReasonerTimeout`, guarding against malformed/oscillating rule sets). Rules are plain Python
  (`Rule(name, antecedents, consequent)`) with simple positional pattern matching — not a general
  unifier, arity is small and fixed.
- **Conflict detection**: since the NC axiom isn't globally enforced, a hand-rolled DPLL SAT solver
  (unit propagation + chronological backtracking + branching, no CDCL/VSIDS — unnecessary at this scale)
  checks consistency of derived norms, **scoped per `(subject, resource)` group** so one unrelated
  conflict can never make the whole system unsatisfiable ("deontic explosion", guarded against
  explicitly). On UNSAT, it extracts the **minimal conflicting subset** (assumption-literal/unsat-core
  style) so a conflict can be explained by which specific norms clash, not just reported as a boolean.
- **Conflict resolution**: a pluggable, named `CombiningAlgorithm` (deny-overrides, permit-overrides,
  first-applicable, priority-weighted — the XACML-standard pattern) replaces the PDF's informally-stated
  "specific beats general, prohibition beats permission." A genuine tie/undecidable case returns "escalate,"
  never a silent guess. Immunity is checked and short-circuits **before** any combining algorithm runs,
  consistent with Hohfeld's own structural priority of immunity over ordinary conflict-weighing.
- **Delegation/liability chain validation**: cycle detection via stdlib `graphlib.TopologicalSorter`
  (delegation graphs must be acyclic — a cycle is either a modeling error or an authority-laundering
  attempt); chain validity itself is a linear walk once each agent has exactly one direct delegator.
- **Condition evaluation**: `Condition(predicate, args)` pairs resolved through a fixed, engine-owned
  predicate registry — **never** `eval()`/`exec()` on data that traces back to an external agent request.
- **Audit trail**: every rule firing and decision is logged (which rule, from which facts, producing
  which derived facts/decision) for explainability.
- **End-to-end pipeline**: query candidates → filter by scope (re-checked at decision time, not cached
  from derivation time — scoped norms can expire between derivation and use) → immunity check → SAT
  consistency check → resolve conflicts (or escalate) → validate delegation chain → decide (permit, with
  derived obligations and liability chain attached; or apply closure policy if nothing matched at all).

Full detail, including the module layout (`models.py`, `rules.py`, `engine.py`, `conditions.py`,
`sat.py`, `resolve.py`, `chains.py`, `pipeline.py`, `audit.py`), Mermaid diagrams, and code sketches, is
in the implementation spec §4–11 — not duplicated here.

## Resolved: stdlib vs. SymPy

Settled, superseding the open question raised earlier in onboarding: **hand-roll everything**, including
the SAT solver — a real DPLL core is on the order of ~240 lines of pure-stdlib Python, and clause sets
here stay small because consistency checks are scoped per `(subject, resource)` group, never global.
SymPy remains the one sanctioned dependency, reserved **only** as a fallback for condition-expression
parsing if the flat predicate registry (`conditions.py`) ever proves insufficient for a real deployment's
condition language (nested boolean expressions, arithmetic) — not used anywhere in the baseline design.

## Requirements and risks already drafted (to seed `/requirements`, not re-derived from scratch)

The implementation spec already contains:

- **Functional requirements FR-1–FR-10** and **non-functional requirements NFR-1–NFR-6** (§12).
- **A risk table R-1–R-10** (§13) covering deontic explosion, bad-inheritance paradoxes (Ross's paradox,
  Good Samaritan paradox), delegation cycles, stale scoped norms, non-termination, condition-injection/
  code execution, undecidable-conflict guessing, priority-governance gaming, and SAT-instance blowup —
  each with its mitigation already specified in the design above.
- **Nine worked test scenarios** (§14.1–14.9), including two historical-paradox regression tests
  (Chisholm's paradox, translated into an agentic scenario, §14.8; deontic-explosion containment, §14.9)
  — these are meant to become the literal contents of the eventual test suite once stories are broken out.

`/requirements` should treat these as the starting draft, adjusting only where the user's own priorities
diverge from the spec's defaults, rather than re-deriving functional/non-functional requirements from
zero.

## Open items still to settle (from the spec's own §15, carried forward — not decided here)

- **Priority governance** (R-9): the spec requires that a norm's `priority` be settable only by the
  granting authority, never by the norm's own subject — but that's an enforcement point for whatever
  stores/authors norms, outside the reasoning engine's own boundary. Worth an explicit requirement that
  the reasoner treats `Norm` objects as coming from a trusted store.
- **Rule-authoring interface**: baseline is Python-authored `Rule` objects (consistent with excluding a
  DSL this iteration, above); a declarative surface syntax compiled to that shape is a plausible later
  addition, not required now.
- **Horty-style prioritized default logic**: `PriorityWeighted` combining algorithm is a pragmatic
  stand-in for the field's most-developed formal treatment of defeasible-obligation prioritization
  (Horty, *Reasons as Defaults*), not a claim of formal equivalence to it.
- **Consistency-check frequency**: baseline checks once per fixed point plus on-demand per resolution
  request; checking after every rule firing would catch conflicts earlier in a long derivation chain at
  higher cost — not decided.
- **SAT solver upgrade path**: if the hand-rolled solver becomes a real bottleneck, the documented
  escalation is to benchmark against `pysat` before considering a dependency change — no performance
  threshold for revisiting this is defined yet.
