# Notes

## Phase
<!-- onboarding | setup | requirements | features | stories | stage-a | stage-b | pr | publish | done -->
<!-- a later iteration (see CLAUDE.md -> New Iterations) re-enters at onboarding/requirements/features and reuses
     these same phase values -- there is no separate "iteration N" phase -->
features

## Project
name: deontic-reasoner
python: 3.14

## Repos
<!-- Filled in during /setup -->
docs (vault root): https://github.com/stefano-bragaglia/deontic-reasoner-docs, visibility: public
code (project/): https://github.com/stefano-bragaglia/deontic-reasoner, visibility: public

## Publish
<!-- Filled in during /publish. Keeps the step resumable after /clear. -->
last published version:
status: <!-- unpublished | built | published | failed -->

## Open Questions
<!-- Add a question when blocked. Clear (delete the line) when answered. -->
<!-- Format: - [ ] Q: <question> / A: <answer> -->

## Features
<!-- status: proposed | approved | branched | stories-merged | pr-open | done -->
<!-- branched: epic branch created lazily by /stage-a on that feature's first story, not by /features -->
<!-- stories-merged: every story for this feature is `merged`, but the epic PR into main isn't open yet -->

## Stories
| Feature | Story | Branch | Status |
|---------|-------|--------|--------|
<!-- status: proposed | approved | tests | code | pr-open | merged -->

## Decisions
<!-- Key choices made and why. Future agents use this to avoid re-litigating. -->
- Added `documentation/references/query-2026-07-21-deontic-reasoner-implementation-spec.md` as the
  primary architectural blueprint for the reasoner, superseding the PDF wherever the two disagree on
  specifics (it was written to formalize the PDF's informal parts under this project's own constraints).
  Resolves the earlier open stdlib-vs-SymPy question: hand-roll everything (incl. a ~240-line DPLL SAT
  solver), SymPy reserved only as a fallback for condition-expression parsing, not used in the baseline.
  Its FR/NFR tables, risk table (R-1–R-10), and nine worked test scenarios are intended to seed
  `/requirements` directly rather than being re-derived from scratch.
- `/setup` complete. Docs repo `deontic-reasoner-docs` and code repo `deontic-reasoner` both created
  public on GitHub per user's explicit choice (setup's own default recommends docs=private; user chose
  public for both). Branch protection applied to code repo's `main` (0 required reviews, CI job `test`
  required + must be up to date, force-push/deletion blocked, `enforce_admins: false`).
- Two deviations from the setup playbook's literal command text, both needed to make the gates actually
  runnable: `radon cc -n B` requires an explicit path argument (added `src tests`), and `pytest` exits
  non-zero on zero collected tests even with the coverage threshold met — added a trivial
  `tests/test_package.py` smoke test (asserts the package imports) so the initial scaffold commit could
  pass its own pre-commit hook without `--no-verify`. Both changes are reflected in
  `project/.git/hooks/pre-commit` and `project/.github/workflows/ci.yml`.
- `uv init --package` seeds a `main()`/`[project.scripts]` CLI entry point by default; removed both
  (this is a library per `Description.md`, no CLI planned for this iteration).
- Requirements Q&A resolved (full detail + rationale in `documentation/Requirements.md → Questions`):
  default combining algorithm is deny-overrides (not an error if omitted); closure policy has no implicit
  default (raises if omitted — safety-relevant, must be explicit per deployment); the four Hohfeldian
  correlative derivations and delegation scope-narrowing are both automatic, built-in engine behavior, not
  opt-in rules; working memory is append-only, no true fact retraction this iteration (revocation is a
  new fact + scope/condition checks, preserving the audit trail and keeping derivation monotonic); the
  implementation spec's nine worked test scenarios (§14.1–14.9) are adopted as-is as acceptance criteria
  for `/features`/`/stories` to map onto.

## Next Action
<!-- One sentence. What should happen next, and who does it (agent or user). -->
Run /features to propose this iteration's epics from documentation/Requirements.md.