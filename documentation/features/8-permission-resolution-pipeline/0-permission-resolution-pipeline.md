# 8. Permission Resolution Pipeline

## Name

`8-permission-resolution-pipeline`

## Summary

The integration feature: the single end-to-end `resolve_request(...)` entry point that composes every
prior feature into the actual permission-decision algorithm — query candidates → filter by scope
(`4-condition-and-scope-evaluation`) → immunity check → consistency check
(`5-conflict-detection`) → resolve conflicts or escalate (`6-conflict-resolution`) → validate delegation
chain (`7-delegation-chain-validation`) → decide. On no matching norm at all, applies the configured
closure policy — which, per Requirements.md → Questions: Default Policies #2, has **no implicit
default**: an engine instance that hasn't configured one raises rather than silently picking permissive
or prohibitive, since that choice is safety-relevant per deployment. This feature also owns the
decision-level half of the audit trail (which norms/algorithm/chain-result led to a given decision — the
engine-level half, which rule fired from which facts, belongs to `2-forward-chaining-engine`). This is
necessarily the last feature: every one of the preceding seven pieces has to exist before there's
anything to orchestrate.

## Requirements covered

- FR 14 — configurable closure policy when no norm matches, no implicit default (raises if unconfigured).
- FR 15 — the full end-to-end pipeline, composing scope filtering, immunity check, consistency check,
  conflict resolution, delegation-chain validation, and decision in that order.
- FR 16 (decision half) — structured audit trail of the decision reached and why.
- Acceptance criteria: implementation spec worked scenario §14.1's second half (falling through to
  closure policy correctly once scope excludes the only candidate norm) and §14.2 (right/duty pair and
  directed obligation — demonstrating the pipeline surfaces a directed obligation's violation, not just
  plain permit/deny).

## Dependencies

- `2-forward-chaining-engine`, `3-hohfeldian-and-delegation-semantics`,
  `4-condition-and-scope-evaluation`, `5-conflict-detection`, `6-conflict-resolution`,
  `7-delegation-chain-validation` — this feature has no content of its own beyond orchestrating all six.

## Stories

See `documentation/features/8-permission-resolution-pipeline/` for the story breakdown (added by
`/stories`).
