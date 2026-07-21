# Notes

## Phase
<!-- onboarding | setup | requirements | features | stories | stage-a | stage-b | pr | publish | done -->
<!-- a later iteration (see CLAUDE.md -> New Iterations) re-enters at onboarding/requirements/features and reuses
     these same phase values -- there is no separate "iteration N" phase -->
stories

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
| Feature | Status | Branch |
|---|---|---|
| 1-core-data-model | approved | |
| 2-preference-ordering-and-best-worlds | approved | |
| 3-obligation-and-permissibility-queries | approved | |
| 4-hohfeldian-grounding | approved | |
| 5-scope-evaluation | approved | |
| 6-forward-chaining-and-delegation | approved | |

## Stories
| Feature | Story | Branch | Status |
|---------|-------|--------|--------|
<!-- status: proposed | approved | tests | code | pr-open | merged -->
| 1-core-data-model | 1-propositional-core-types | | approved |
| 1-core-data-model | 2-hohfeldian-norm-and-relation | | approved |
| 1-core-data-model | 3-json-round-trip | | approved |
| 2-preference-ordering-and-best-worlds | 1-rule-violation-and-hard-constraint-exclusion | | approved |
| 2-preference-ordering-and-best-worlds | 2-preference-ordering-three-criteria | | approved |
| 2-preference-ordering-and-best-worlds | 3-best-worlds-with-scoped-enumeration | | approved |

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
- Requirements Q&A resolved (full detail + rationale in `documentation/Requirements.md → Questions`):
  `World = frozenset[str]`, with `best_worlds` enumeration scoped to the atoms actually mentioned in the
  loaded rule set/antecedent (this vault's own extrapolation, not a sourced claim); `Rule.weight` gets
  the same trust-boundary treatment `Norm.priority` had (granting authority sets it, never the rule's
  own subject); `scope_matches` reuses the predicate-registry design unchanged (trust boundary is
  independent of which deontic framework is underneath); hard constraints stay in the requirement set
  but are unexercised reserve infrastructure this iteration; the implementation spec's nine worked
  scenarios (§14.1–14.9) are adapted (not dropped, not rewritten from scratch) to weighted-count
  preference semantics, scenario-by-scenario; powers need no dedicated predicate (an ordinary
  domain-specific fact as a conditional rule's body suffices, generalizing delegation's own mechanism);
  `Relation.IMMUNITY` is purely representational this iteration, no short-circuit behavior yet.
- `Description.md`'s own `## Open questions` section (4 items, a subset of `Requirements.md`'s 7) now
  also carries `_A:_` answers in place, matching the resolutions already recorded in `Requirements.md` —
  kept both documents internally consistent rather than leaving `Description.md`'s questions dangling
  unanswered after `Requirements.md` settled them.
- Six features approved (`documentation/features/1-core-data-model` through
  `6-forward-chaining-and-delegation`), numbered in dependency/build order — notably leaner than the
  previous (superseded) 8-feature breakdown, since no dedicated SAT-detection/conflict-resolution
  feature is needed (weighted-count preference in feature 2 handles that directly). All nine adapted
  worked scenarios (§14.1–14.9) are assigned across features 3, 4, 5, and 6. No epic branches yet —
  created lazily by `/stage-a` per `CLAUDE.md → Branching Model`.
- `1-core-data-model` broken into 3 approved stories (propositional core types, Hohfeldian
  norm/relation, JSON round-trip) — deliberately data-only, no evaluation behavior (`violates`,
  hard-constraint exclusion) leaks in here; that's `2-preference-ordering-and-best-worlds`'s job.
  `Rule.head` is a `frozenset[Atom]` (conjunction), not a single `Atom` like the interaction model's
  illustrative example — needed by a couple of the adapted worked scenarios, AND-only, no formula parser.
- `2-preference-ordering-and-best-worlds` broken into 3 approved stories (violation/exclusion,
  three-criteria preference ordering, scoped best-worlds enumeration). Non-obvious catch recorded in
  story 2: `WEIGHTED_COUNT`'s weight summation must iterate in a stable, sort-key-based order (not raw
  `frozenset` iteration order), since Python's per-process string-hash randomization could otherwise
  make float-summation rounding vary between separate runs of the same program — a subtle violation of
  NFR 4 (determinism) that pure "same output for same input within one run" testing wouldn't catch.

## Next Action
<!-- One sentence. What should happen next, and who does it (agent or user). -->
Run /stories 3-obligation-and-permissibility-queries next (remaining: 3 through
6-forward-chaining-and-delegation) — /stage-a does not start until every feature has an approved story
breakdown.