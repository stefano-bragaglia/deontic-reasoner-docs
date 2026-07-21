# 5. Conflict Detection

## Name

`5-conflict-detection`

## Summary

Since this reasoner deliberately does not globally enforce Standard Deontic Logic's "obligations cannot
conflict" axiom (Description.md → *Deontic operators, and a deliberate departure from Standard Deontic
Logic*), a derived norm set genuinely can be inconsistent — both `O(A does X)` and `F(A does X)` for the
same act. Detecting that is a Boolean satisfiability question. This feature is a hand-rolled DPLL SAT
solver (unit propagation, chronological backtracking, static branching order — deliberately no
CDCL/VSIDS, unnecessary at this scale) with assumption literals and unsat-core extraction, so a detected
conflict can be explained by *which* specific norms clash, not just reported as a boolean. Consistency
checks are scoped per `(subject, resource)` group — never a single global check — so one unrelated
conflict elsewhere in the system can never make an unrelated decision unsatisfiable ("deontic
explosion").

## Requirements covered

- FR 8 — detect joint inconsistency in a derived norm set, scoped per `(subject, resource)` group.
- FR 9 — report the minimal explaining subset of norms (unsat core) on conflict, not just a boolean.
- NFR 5 — consistency checks scoped, not global; per-request latency independent of total norm count.
- Acceptance criteria: implementation spec worked scenario §14.9 (deontic explosion containment — a
  conflict in one `(subject, resource)` group must not affect an unrelated group's own consistency
  check).

## Dependencies

- `1-core-data-model` — norms are encoded as literals keyed by `(subject, action, resource, status)`, so
  `Norm`/`DeonticStatus` must exist. Does not depend on the engine or domain rules — this is a
  general-purpose SAT module operating on whatever norms it's handed.

## Stories

See `documentation/features/5-conflict-detection/` for the story breakdown (added by `/stories`).
