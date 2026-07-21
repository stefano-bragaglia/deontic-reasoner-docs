# 4. Delegation Obligations — Audit, Dyadic Revoke-on-Violation, Liability

## What it does

The second built-in rule triggered by the same `delegate` fact as
`3-delegation-grant-with-scope-narrowing`: derives the delegator's oversight obligations — an audit
duty, a **dyadic** (conditional) revoke-on-violation duty, and a liability fact — exactly the
`delegate(A,B,perm) → O(A,audit) ∧ O(A,revoke|violation) ∧ liable(A,...)` pattern from Description.md and
the implementation spec.

## Inputs / outputs

Continuing the `3-delegation-grant-with-scope-narrowing` example
(`Fact("delegate", ("AnalyticsAgent", "VisualizationAgent", "c1", "p1"))`), firing this rule derives:

- An audit-duty `Norm`: `status=DeonticStatus.OBLIGATORY`, `subject="AnalyticsAgent"`,
  `action="audit"`, `resource` = the parent norm's resource, `given=None` (unconditional).
- A revoke-on-violation `Norm`: `status=DeonticStatus.OBLIGATORY`, `subject="AnalyticsAgent"`,
  `action="revoke"`, `resource` = the parent norm's resource,
  `given=Condition("violation", ("VisualizationAgent", "p1"))` — **dyadic**, not unconditional.
- `Fact("liable", ("AnalyticsAgent", "VisualizationAgent", "p1"))`.

## Edge cases and failure modes

- **The dyadic condition is representation only, not evaluation.** This story creates the revoke norm
  *with* `given=Condition("violation", (B, PID))` — it does **not** decide whether a violation has
  actually occurred, and does not itself derive an unconditional "must revoke now" fact. Actually
  evaluating whether `given` currently holds (against a `violation` fact that may or may not be present)
  is `4-condition-and-scope-evaluation`'s job, at decision time — conflating the two is exactly the
  Chisholm's-paradox failure mode (§14.8) this design is built to avoid. This story's own tests verify
  only that the *conditional* norm object is created correctly, not that it "fires" — full end-to-end
  Chisholm-paradox verification (§14.8) is deferred to whichever of `4-condition-and-scope-evaluation` or
  `8-permission-resolution-pipeline`'s stories actually evaluates `given` against facts.
- **Determinism**: like the grant norm, the audit/revoke norms' `id`s must be deterministic functions of
  `(A, B, PID)` (e.g. `f"audit:{A}:{B}:{PID}"`, `f"revoke:{A}:{B}:{PID}"`) — re-asserting the identical
  `delegate` fact must never produce a second pair of differently-`id`'d obligation norms.
- **Missing parent**: a `delegate` fact whose `PID` isn't in `engine.norms` derives none of the three
  outputs (dangling reference, not an error — same convention as story 3).
- **Independent of the grant rule's own success**: if `3-delegation-grant-with-scope-narrowing`'s rule
  didn't fire (e.g. `parent.delegable=False`), this rule fires independently off the same `delegate`
  fact regardless — the audit/revoke/liability obligations on the delegator are about the *attempt to
  delegate*, not conditional on whether a new norm actually got granted to `B`. (Whether that's the
  right call given a non-delegable parent is worth a second look once this is wired into
  `8-permission-resolution-pipeline`, but nothing in `Requirements.md` says oversight obligations should
  be skipped for a rejected delegation attempt, so this story keeps the two rules fully independent.)

## Acceptance criteria

1. `build_delegation_obligations_rule()` returns a `Rule` with antecedent
   `("delegate", (Var("A"), Var("B"), Var("CID"), Var("PID")))` (same shape as story 3's rule, a
   different, independent rule).
2. Firing it registers an audit-duty norm (`OBLIGATORY`, `subject=A`, `action="audit"`,
   `resource=parent.resource`, `given=None`) via `register_norm`.
3. Firing it registers a revoke-on-violation norm (`OBLIGATORY`, `subject=A`, `action="revoke"`,
   `resource=parent.resource`, `given=Condition("violation", (B, PID))`).
4. Firing it yields `Fact("liable", (A, B, PID))`.
5. The audit and revoke norms' `id`s are deterministic: re-asserting the identical `delegate` fact and
   re-running never produces a second pair with different ids.
6. A `delegate` fact with a missing `PID` derives none of the above.
7. The revoke norm's presence in `engine.norms` does **not** by itself imply
   `Fact("active", ...)`/any unconditional revoke obligation exists — only the conditioned `Norm` object
   exists; nothing in this story's own test suite asserts an actual revoke was "triggered."

## Tasks

1. Add `build_delegation_obligations_rule()` to `hohfeld.py`, using `register_norm`
   (`3-delegation-grant-with-scope-narrowing`).
2. Extend `create_reasoner_engine` (`2-engine-factory-with-builtins`) to always include this rule too.
3. Write `tests/test_hohfeld_delegation_obligations.py` covering acceptance criteria 2–7 — this is the
   direct implementation of worked scenario §14.3.

## Deliverables

- `project/src/deontic_reasoner/hohfeld.py` (modified)
- `project/tests/test_hohfeld_delegation_obligations.py` (new)

## Dependencies

- `1-norm-registration-and-correlative-rule`, `2-engine-factory-with-builtins`,
  `3-delegation-grant-with-scope-narrowing` (shares its `register_norm` helper and antecedent shape).
