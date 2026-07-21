# 6. Conflict Resolution

## Name

`6-conflict-resolution`

## Summary

What happens once `5-conflict-detection` reports a conflict: a named, swappable `CombiningAlgorithm`
(deny-overrides, permit-overrides, first-applicable, priority-weighted — the XACML-standard pattern),
selectable per domain/resource-class rather than hardcoded. Per Requirements.md → Questions: Default
Policies #1, if a call site doesn't specify one, the system defaults to **deny-overrides** rather than
raising — an unspecified conflict resolves toward denial, never silently toward access. This feature also
owns the **immunity short-circuit**: an `IMMUNITY` norm is checked and blocks *before* any combining
algorithm runs at all, since Hohfeld's own structural priority means immunity isn't "a very strong
permission" to be weighed by priority — it structurally disables the relevant `POWER`-derived norm. And
it owns **escalation**: when a combining algorithm returns "undecidable" (e.g. a genuine priority tie),
the system surfaces that explicitly rather than guessing.

## Requirements covered

- FR 10 — resolve conflicts via a named, swappable combining algorithm, with deny-overrides as the
  default when unspecified (Requirements.md → Questions: Default Policies #1).
- FR 11 — immunity checked and short-circuits before any combining algorithm runs.
- FR 12 — escalate (explicit "unresolved" result) rather than guess on a genuine tie/undecidable case.
- Acceptance criteria: implementation spec worked scenarios §14.4 (conflict detection and deny-overrides
  resolution) and §14.6 (power exercise creates a new norm; immunity blocks its revocation even against
  a hypothetical permit-overrides policy).

## Dependencies

- `1-core-data-model` — `Norm.priority`, `Relation.IMMUNITY` must exist.
- `5-conflict-detection` — a combining algorithm resolves the specific conflicting set that conflict
  detection identifies; nothing to resolve without it.

## Stories

See `documentation/features/6-conflict-resolution/` for the story breakdown (added by `/stories`).
