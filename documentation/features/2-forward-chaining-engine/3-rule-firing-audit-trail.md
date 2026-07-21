# 3. Rule-Firing Audit Trail

## What it does

Extends the fixed-point loop to record, for every successful rule firing, which rule fired, the
bindings that satisfied it, and the facts it produced — exposed on `RunResult` so a caller (and later,
the resolution pipeline in `8-permission-resolution-pipeline`) can explain *why* a fact was derived, not
just that it was.

## Inputs / outputs

- `result.audit_log: tuple[AuditEntry, ...]`, each entry naming the rule, the bindings used, and the
  facts produced by that specific firing — in the same deterministic firing order as
  `2-fixed-point-loop-and-termination`'s acceptance criterion 6.

## Edge cases and failure modes

- A rule that matches and fires but whose consequent only yields facts already known (no new
  information) still gets an audit entry — the audit trail records every successful *match+fire*, not
  only the subset that contributed new facts. This is a deliberate reading of FR16 ("which rule fired,
  from which facts, producing which derived facts") — "this rule fired here and reconfirmed X" is itself
  useful explanatory information, and dropping reconfirming firings from the log would make it look like
  a rule silently stopped applying once its consequent's output became "old news."
- Audit entries appear in the same deterministic order as fact derivation, iteration by iteration — not
  grouped by rule, not sorted after the fact.
- A rule that matches zero times in a run contributes zero audit entries (never a placeholder "no
  match" entry).
- Two runs of the same engine setup must produce identical `audit_log` sequences (content and order),
  consistent with NFR-3's determinism guarantee — the audit trail must not itself become a source of
  nondeterminism.

## Acceptance criteria

1. `AuditEntry` (new type) carries at minimum: the firing rule's name, the `Bindings` that satisfied it,
   and the tuple of `Fact`s it produced.
2. `RunResult.audit_log: tuple[AuditEntry, ...]`.
3. A single-rule, single-firing scenario produces exactly one `audit_log` entry, naming that rule and
   its produced facts.
4. A rule that fires against multiple independent binding sets in one run produces one `audit_log` entry
   per (rule, bindings) firing — not one entry per rule regardless of how many times it fired.
5. A rule firing that reconfirms an already-known fact still produces an audit entry (see edge case
   above).
6. Two runs of an identical engine setup produce identical `audit_log` sequences, both in content and in
   order.

## Tasks

1. Define `AuditEntry` in `src/deontic_reasoner/engine.py`.
2. Extend `ReasonerEngine.run()` to append an `AuditEntry` per successful rule firing, in the same
   deterministic order fact derivation already uses.
3. Extend `RunResult` with `audit_log`.
4. Write `tests/test_engine_audit_log.py` covering acceptance criteria 3–6.

## Deliverables

- `project/src/deontic_reasoner/engine.py` (modified)
- `project/tests/test_engine_audit_log.py` (new)

## Dependencies

- `2-fixed-point-loop-and-termination`.
