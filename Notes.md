# Notes

## Phase
<!-- onboarding | setup | requirements | features | stories | stage-a | stage-b | pr | publish | done -->
<!-- a later iteration (see CLAUDE.md -> New Iterations) re-enters at onboarding/requirements/features and reuses
     these same phase values -- there is no separate "iteration N" phase -->
requirements

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
- [ ] Q: Are the four Hohfeldian correlative derivations auto-applied built-in rules, or opt-in building blocks? (#3)
- [ ] Q: Is scope-narrowing on re-delegation enforced automatically by the engine, or opt-in? (#4)
- [ ] Q: Is true fact retraction needed this iteration, or is append-only (spec's model) sufficient? (#5)
- [ ] Q: Adopt the implementation spec's nine worked test scenarios (§14.1–14.9) as acceptance criteria as-is? (#6)

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

## Next Action
<!-- One sentence. What should happen next, and who does it (agent or user). -->
User to answer the six open questions in documentation/Requirements.md (also mirrored in Open Questions
above); once resolved, set Phase: features and run /features.