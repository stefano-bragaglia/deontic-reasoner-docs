# Requirements

Derived from `documentation/Description.md`, which is itself grounded in
`documentation/references/Deontic Logic for Agent Permissions - A Formal Framework for AI Agent
Governance.pdf` (the motivating essay) and
`documentation/references/query-2026-07-21-deontic-reasoner-implementation-spec.md` (the primary
architectural blueprint — cited below as "the implementation spec"). Where a requirement below
corresponds to an FR/NFR/R-id already drafted in the implementation spec, that id is noted in
parentheses so the two documents stay traceable to each other.

## Functional Requirements

1. The system shall represent agents, resources, facts, and norms as first-class, JSON-serializable
   data — `frozen=True, slots=True` dataclasses round-tripping through `dataclasses.asdict`/a small
   `from_dict`, no custom binary format, no ORM or database dependency. (spec FR-1, NFR-6)
2. The system shall represent all four Hohfeldian relations (privilege, right, power, immunity) as a
   first-class `Relation` enum on every norm, alongside the norm's deontic status (obligatory,
   permitted, forbidden). The engine shall **automatically and always** derive each relation's
   correlative (Right → counterparty Duty, Privilege → No-Right, Power → Liability, Immunity →
   Disability) as a built-in forward-chaining rule whenever a norm of that relation type is asserted — no
   per-domain rule-authoring is required to get the correlative, since correlativity is a definitional
   consequence of Hohfeld's framework, not an optional convenience. (spec FR-1; see *Questions: Built-in
   Semantics* #3)
3. The system shall support conditional (dyadic) obligations — `O(φ | ψ)` as a primitive, not a material
   conditional — so contrary-to-duty norms (e.g. "revoke access on violation") can be represented and
   reasoned about without the classical contrary-to-duty failure modes (Chisholm's paradox). (spec §3.3)
4. The system shall forward-chain from a set of ground facts and norms to a fixed point: matching rules
   against working memory, firing them, collecting newly derived facts/norms, and repeating until no new
   facts are derived or a `max_iterations` cap is hit (raising an explicit exception rather than
   hanging). (spec FR-2, NFR-4) Working memory is **append-only**: this iteration does not require true
   fact retraction. Revocation and expiry are represented by asserting new facts (e.g. a `revoked(...)`
   fact) and checking scope/conditions at decision time, never by deleting a previously-asserted fact —
   this both preserves the audit trail (req. 16) and keeps derivation monotonic, which the termination
   argument (NFR-4) relies on. (see *Questions: Built-in Semantics* #5)
5. The system shall derive the correlative obligations of a delegation grant automatically — audit duty,
   dyadic revoke-on-violation duty, and liability — from a single `delegate(A, B, permission)` fact, with
   no manual authoring of the derived norms. (spec FR-2, §6.2) Core delegation handling shall also
   automatically enforce scope-narrowing: a re-delegated scope is intersected with the delegator's own
   scope, so a delegatee can never end up with a wider scope than its delegator held — this is built-in
   engine behavior, not an opt-in rule a domain must remember to add. (spec §14.7; see *Questions:
   Built-in Semantics* #4)
6. The system shall support scoped norms (temporal validity, execution context, registered conditions)
   and evaluate scope at **decision time**, not only at derivation time, since a norm can expire between
   being derived and being queried. (spec FR-3, R-5)
7. The system shall evaluate `Condition` objects only through a fixed, engine-owned predicate registry —
   never via `eval()`/`exec()` or any other execution of agent-supplied text as code. (spec NFR-2, R-7)
8. The system shall detect when a derived set of norms touching the same `(subject, resource)` is jointly
   inconsistent, via a satisfiability check scoped to that group — never a single global check — so one
   unrelated conflict can never make an unrelated decision unsatisfiable ("deontic explosion"). (spec
   FR-4, NFR-5, R-1)
9. On detecting an inconsistency, the system shall report the **minimal explaining subset of norms**
   (an unsat core) responsible for the conflict, not merely a true/false result. (spec FR-6)
10. The system shall resolve a detected conflict via a named, swappable combining algorithm — at minimum
    deny-overrides, permit-overrides, first-applicable, and priority-weighted — selectable per
    domain/resource-class rather than hardcoded globally. If a call site doesn't specify one, the system
    shall default to **deny-overrides** (matching the source PDF's "prohibition beats permission") rather
    than raising an error — an unspecified conflict resolves toward denial, not silently toward access.
    (spec FR-5, §8.1; see *Questions: Default Policies* #1)
11. The system shall check immunity **before** invoking any combining algorithm: an immunity-protected
    normative position can never be overridden by ordinary conflict resolution, regardless of what a
    combining algorithm would otherwise decide. (spec §8.2)
12. When a combining algorithm cannot resolve a conflict (a genuine tie or otherwise undecidable case),
    the system shall escalate (return an explicit "unresolved" result naming the conflicting candidates)
    rather than pick an arbitrary winner. (spec FR-9, R-8)
13. The system shall validate delegation/liability chains: detect cycles structurally (a delegation graph
    must be acyclic) and detect broken links (a grantor that no longer actually holds the power it
    purportedly delegated), denying the dependent request regardless of the leaf permission's own
    validity. (spec FR-7, R-4)
14. The system shall apply a configurable closure policy — permissive (unaddressed ⇒ permitted) or
    prohibitive (unaddressed ⇒ forbidden) — when no norm matches a request at all. There is **no
    implicit global default**: if a deployment/engine instance doesn't configure a closure policy, the
    system shall raise rather than silently pick one, since permissive vs. prohibitive is a
    safety-relevant choice each deployment must make explicitly. (spec FR-8, §3.4; see *Questions:
    Default Policies* #2)
15. The system shall expose an end-to-end permission-resolution pipeline that composes the above in
    order: query candidates → filter by scope → immunity check → consistency check → resolve conflicts
    (or escalate) → validate delegation chain → decide, returning `PERMIT` (with attached obligations and
    liability chain) or `DENY` (with a reason), or an escalation. (spec §10)
16. The system shall produce a structured audit trail recording, at minimum: which rule fired, from
    which facts, and what it derived; and for each resolution, the decision reached and why (which norms,
    which combining algorithm, which chain validation result). (spec FR-10)

## Non-Functional Requirements

1. **Platform**: Python 3.14. Standard library only for the reasoning engine's baseline. (spec NFR-1)
2. **Dependency ceiling**: SymPy is the one permitted optional dependency, reserved solely as a fallback
   for condition-expression parsing (`conditions.py`) if the flat predicate registry proves insufficient
   — not used anywhere in the baseline implementation. No Prolog/ASP bindings, no external process or
   database dependency, no other third-party logic/rule-engine library. (spec NFR-1, constraint in §2)
3. **No dynamic code execution** on data that traces back to an external agent request — no
   `eval()`/`exec()`/`ast.literal_eval` on agent-supplied expressions. (spec NFR-2, R-7)
4. **Determinism**: given the same facts, norms, and rule set, the engine shall produce the same fixed
   point and the same conflict/resolution outcome on every run. No observable behavior may depend on
   dict/set iteration order — anything order-sensitive (e.g. rule firing order) must be explicitly
   sorted. (spec NFR-3)
5. **Bounded termination**: the fixed-point loop is always capped by `max_iterations`; a rule set that
   never reaches a fixed point fails loudly (a raised exception), never hangs. (spec NFR-4, R-6)
6. **Scoped cost**: consistency checks are scoped per `(subject, resource)` group, so per-request latency
   does not grow with total system-wide norm count. (spec NFR-5)
7. **JSON round-trip**: every core dataclass serializes to and from plain JSON with no custom encoder
   beyond `datetime.isoformat()`/enum `.value`. (spec NFR-6)
8. **Trust boundary**: the reasoning engine treats `Norm` objects (including their `priority` field) as
   coming from a trusted store; enforcing that only a granting authority — never a norm's own subject —
   can set or raise a norm's priority is the responsibility of whatever system stores/authors norms, not
   the reasoning engine itself. (spec R-9)
9. **Test coverage**: ≥90% coverage, both aggregate and per-file, per this vault's standing project-wide
   quality gate (`CLAUDE.md → Hard Rules`; enforced by `project/scripts/check_file_coverage.py`).

## User Interaction Model

This iteration ships **a Python library only** — no CLI, no server, no persistence layer, no textual
DSL. The "user" is a developer (or, in a later iteration, an MCP server) embedding the reasoner directly
in Python code: constructing facts/norms, running the engine, and calling the resolution pipeline.

Illustrative (not final API — exact signatures are a Stage A/B implementation detail):

```python
from deontic_reasoner import (
    Agent, Norm, Fact, Relation, DeonticStatus, Scope, Condition,
    ReasonerEngine, Rule, Var,
    resolve_request, DenyOverrides,
)

# 1. Describe the world: agents, norms, rules.
engine = ReasonerEngine(rules=[DELEGATION_OBLIGATIONS_RULE], closure=DeonticStatus.PERMITTED)
engine.facts.add(Fact("delegate", ("Orchestrator", "DataAgent", "R1")))

# 2. Run forward chaining to a fixed point.
result = engine.run()
# result.facts now includes the derived audit/revoke-on-violation/liability norms.

# 3. Ask a permission question.
decision = resolve_request(
    engine,
    agent="DataAgent", action="read", resource="/data/analytics/report.csv",
    context={"purpose": "reporting"},
    combining_algorithm=DenyOverrides(),
)

if decision.permitted:
    print(decision.obligations, decision.liability_chain)
elif decision.escalated:
    print("needs human review:", decision.conflicting_norms)
else:
    print("denied:", decision.reason)
```

Every `Norm`/`Fact`/`Agent` is also expected to round-trip through JSON (`dataclasses.asdict`/a small
`from_dict`), so a caller can author norms as plain JSON/dict literals rather than constructing dataclass
instances by hand, if that's more convenient for their integration.

## Questions: Default Policies

1. *Q: When a caller doesn't specify a combining algorithm for a `(subject, resource)` conflict group,
   should the engine apply a system-wide default (e.g. deny-overrides, matching the source PDF's
   "prohibition beats permission"), or should every call site be required to specify one explicitly
   (raising an error if omitted)?*
   _A: System default is deny-overrides, matching the source PDF's "prohibition beats permission." A
   caller may still override it per call/domain, but omitting it is not an error — it resolves toward
   denial._
2. *Q: When a caller doesn't specify a closure policy for a request, should the engine require it
   explicitly every time (raise if omitted, since the implementation spec's `ReasonerEngine.closure`
   defaults to `None` meaning "must be explicit"), or should there be a global fallback default — and if
   so, permissive or prohibitive?*
   _A: Raise if omitted — no implicit default. Closure has real safety consequences (a general-purpose
   assistant and a money-moving agent need opposite defaults), so every deployment must state its choice
   explicitly rather than silently inheriting one._

## Questions: Built-in Semantics

3. *Q: Should the four Hohfeldian correlative derivations (Right → Duty, Privilege → No-Right, Power →
   Liability, Immunity → Disability) be automatic, built-in derivation rules the engine always applies
   whenever a norm of that relation type is asserted, or building blocks a rule-author must explicitly
   wire up per domain (as the implementation spec's one worked example — the delegation-obligations rule
   — does)?*
   _A: Automatic and built-in — the engine always derives the correlative fact whenever a norm of that
   relation type is asserted, with no per-domain opt-in required. The four correlatives are definitional
   consequences of Hohfeld's own framework, not an optional convenience, so no domain should be able to
   forget to wire one up._
4. *Q: Should scope-narrowing on re-delegation (a delegatee's granted scope must be a subset of the
   delegator's own scope, never wider — worked example §14.7 in the implementation spec) be enforced
   automatically by the engine's core delegation handling, or left as an example/opt-in rule?*
   _A: Automatic and built-in, consistent with the correlatives decision (#3) — core delegation handling
   always intersects a re-delegated scope with the delegator's own, rejecting or narrowing any widened
   re-delegation, with no per-domain opt-in required._
5. *Q: Is true fact retraction (actually removing a previously-asserted fact from working memory) needed
   in this iteration, or is the implementation spec's append-only model sufficient — i.e. revocation and
   expiry are represented by new facts plus time/condition-scoping rather than deleting old facts (which
   also keeps the audit trail intact)?*
   _A: Minimal — append-only, no true retraction. This isn't a simplification made at the expense of
   correctness: append-only **is** the correct semantics for permission governance. Revoking access must
   not erase the fact that access was once granted — both "granted at T1" and "revoked at T2" need to
   coexist for the audit trail (req. 16) to mean anything. True retraction would only be needed for a
   genuinely different feature — answering "what would today look like if a past grant had never
   happened," a retroactive/counterfactual rule-change question that belongs to legal-history-style
   reasoning, not agent-permission governance, and isn't required here. If it's ever needed, the cheap
   escape hatch is constructing a fresh `ReasonerEngine` from a corrected fact set and re-running it,
   rather than building live retraction into a running engine — which would also break the fixed-point
   termination argument's monotonicity assumption (spec §6.3)._

## Questions: Testing Scope

6. *Q: Should the nine worked test scenarios already drafted in the implementation spec (§14.1–14.9,
   including the Chisholm's-paradox and deontic-explosion-containment regression tests) be adopted
   more-or-less verbatim as this iteration's acceptance criteria, or does the breakdown into
   features/stories need to reshape any of them?*
