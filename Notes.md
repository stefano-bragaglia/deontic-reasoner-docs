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
- [ ] Q: Are hard constraints actually needed by this iteration's scenarios? (#4)
- [ ] Q: Which worked scenarios ground this iteration's acceptance criteria? (#5)
- [ ] Q: How is a power's exercise represented as a triggering fact? (#6)
- [ ] Q: Does Relation.IMMUNITY need any special semantic treatment this iteration? (#7)

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
- Added `documentation/references/query-2026-07-21-deontic-reasoner-implementation-spec.md` as an
  architectural reference for the reasoner. **Superseded** (see pivot below) for its SAT-based
  conflict-detection/combining-algorithm design specifically; its dataclass/JSON data-modeling
  conventions and forward-chaining fixed-point loop mechanics remain valid implementation patterns.
- **Framework pivot**: adopted `documentation/references/query-2026-07-21-modern-deontic-framework-minimal-reasoner.md`
  as the primary theoretical/scope blueprint, at the user's request to go minimal. Core framework is now
  **Preferential (Betterness-Ordering) Dyadic Deontic Logic** (Hansson dyadic `O(q|p)` + KLM preferential
  semantics) — a 13-predicate minimal set (9 propositional core: atoms/worlds/conditional
  rules/hard constraints/violates/preferred/best_worlds/obligation query/permissibility query; 4 agentic
  extension: norm/counterparty/delegated/scope_matches), with **weighted-count** as the default
  preference criterion. This *replaces* the earlier SAT-based conflict detection, unsat-core extraction,
  XACML-style combining algorithms, and `graphlib` delegation-chain validation — conflict resolution now
  falls directly out of `best_worlds` under the weighted-count ordering, no separate machinery needed.
  Forward chaining stays in scope (for norm-generating power exercises and delegation's derived
  obligations) but is layered around this new query engine, not around SAT. This also makes the earlier
  stdlib-vs-SymPy SAT question moot — there's no SAT solving in the core at all now; SymPy could only
  ever resurface for the `scope_matches` condition language, same as before.
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
- User correction (still valid, applies to whatever the next iteration through this project looks
  like): `/stage-a` must not start until **every** feature has an approved story breakdown, not just the
  first one worked. See global memory `feedback_stories-before-stage-a`.
- **Reset**: at the user's explicit request, deleted `documentation/Requirements.md` and all of
  `documentation/features/` (8 approved epics and 19 approved stories across them) to go back and revise
  `Description.md` first. Phase reset to `onboarding`. `documentation/references/` (the PDF, the
  implementation spec, `External-Links.md`) and `project/`'s scaffold from `/setup` are untouched — only
  the requirements/features/stories layer was removed, since it all derives from the description that's
  now being revised. The prior decisions above (SymPy-vs-stdlib resolution, `/setup` specifics) remain
  valid facts independent of this reset; the Requirements Q&A and feature/story-specific decisions that
  were here previously were removed since they referenced documents that no longer exist — see git
  history on the docs repo if that reasoning is ever needed again.
- `Description.md` re-synthesized via `/describe` into the skill's required structure (Purpose/Inputs
  and outputs/Key behaviours/Out of scope/Open questions/Tech stack) and approved by the user as-is, no
  changes requested. Content is the same minimal Preferential Dyadic Deontic Logic framework from the
  prior pivot, now explicit that the MCP server is deferred to a separate later iteration.

## Next Action
<!-- One sentence. What should happen next, and who does it (agent or user). -->
User to answer the seven open questions in documentation/Requirements.md (also mirrored in Open
Questions above, asked one at a time); once resolved, set Phase: features and run /features.