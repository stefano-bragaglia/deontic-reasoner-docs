# Notes

## Phase
<!-- onboarding | setup | requirements | features | stories | stage-a | stage-b | pr | publish | done -->
<!-- a later iteration (see CLAUDE.md -> New Iterations) re-enters at onboarding/requirements/features and reuses
     these same phase values -- there is no separate "iteration N" phase -->
stage-a

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
| 1-core-data-model | done | feature/1-core-data-model |
| 2-preference-ordering-and-best-worlds | branched | feature/2-preference-ordering-and-best-worlds |
| 3-obligation-and-permissibility-queries | approved | |
| 4-hohfeldian-grounding | approved | |
| 5-scope-evaluation | approved | |
| 6-forward-chaining-and-delegation | approved | |

## Stories
| Feature | Story | Branch | Status |
|---------|-------|--------|--------|
<!-- status: proposed | approved | tests | code | pr-open | merged -->
| 1-core-data-model | 1-propositional-core-types | (merged, branch deleted) | merged |
| 1-core-data-model | 2-hohfeldian-norm-and-relation | (merged, branch deleted) | merged |
| 1-core-data-model | 3-json-round-trip | (merged, branch deleted) | merged |
| 2-preference-ordering-and-best-worlds | 1-rule-violation-and-hard-constraint-exclusion | (merged, branch deleted) | merged |
| 2-preference-ordering-and-best-worlds | 2-preference-ordering-three-criteria | (merged, branch deleted) | merged |
| 2-preference-ordering-and-best-worlds | 3-best-worlds-with-scoped-enumeration | story/2-preference-ordering-and-best-worlds/3-best-worlds-with-scoped-enumeration | tests |
| 3-obligation-and-permissibility-queries | 1-obligation-and-permissibility-queries | | approved |
| 3-obligation-and-permissibility-queries | 2-weighted-conflict-resolution-regression | | approved |
| 3-obligation-and-permissibility-queries | 3-chisholm-paradox-regression | | approved |
| 3-obligation-and-permissibility-queries | 4-deontic-explosion-containment-regression | | approved |
| 4-hohfeldian-grounding | 1-norm-atom-and-correlative-rule-grounding | | approved |
| 4-hohfeldian-grounding | 2-directed-obligation-regression | | approved |
| 5-scope-evaluation | 1-predicate-registry-and-condition-evaluation | | approved |
| 5-scope-evaluation | 2-temporal-scope-evaluation | | approved |
| 6-forward-chaining-and-delegation | 1-delegation-obligations-grounding | | approved |
| 6-forward-chaining-and-delegation | 2-power-exercise-norm-generation | | approved |
| 6-forward-chaining-and-delegation | 3-delegation-grant-with-scope-narrowing | | approved |
| 6-forward-chaining-and-delegation | 4-forward-chaining-fixed-point-loop | | approved |

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
- `3-obligation-and-permissibility-queries` broken into 4 approved stories (core queries, plus one
  test-only regression story each for §14.4, §14.8, §14.9). Deliberate vacuous-truth semantics
  documented in story 1: on an empty `best_worlds` result, `is_obligatory` is `True`, `is_permitted` is
  `False` (classical universal/existential quantification over an empty set).
- `4-hohfeldian-grounding` broken into 2 approved stories (norm/correlative-rule grounding, directed-
  obligation regression for §14.2). Correlativity is a high-weight ordinary `Rule`, deliberately **not**
  a `HardConstraint` — a hard constraint would make it impossible to ever represent a duty being
  violated, defeating the point. `PRIVILEGE`'s own correlative (`no_right`) is scoped out — unexercised
  by any of the nine adapted scenarios (YAGNI).
- `5-scope-evaluation` broken into 2 approved stories (predicate registry/condition evaluation,
  temporal scope + §14.1 regression). Clarified a mismatch between Requirements Q3's "(predicate_name,
  args) pairs" phrasing and feature 1's already-approved `Norm.condition: Atom | None` (a bare name, no
  separate args field): registered callables take `(norm, request)` directly instead of a separate
  `args` tuple — same no-`eval()`/`exec()` safety property, just no missing field.
- `6-forward-chaining-and-delegation` (last feature) broken into 4 approved stories (delegation
  grounding, power-exercise generation, delegation narrowing, the integration loop). Every feature now
  has an approved story breakdown — 18 stories total across 6 features. Three findings worth
  remembering: (1) the dyadic revoke-on-violation condition needs no new type, it's just the violation
  atom folded into the rule's `body` conjunction; (2) §14.5's "permission fails to derive" reads as
  `is_obligatory` going `True`→`False`, not `is_permitted`→`False` — an unconstrained atom is permitted
  by default in this framework, a real and deliberate property, not a gap; (3) genuine multi-round
  fixed-point convergence is made real, not just claimed, by grounding every norm *before* applying
  power exercises within each iteration, so a newly-exercised norm's own correlative is deferred to the
  next round — tested explicitly in story 4.
- **Quality gate added, at the user's request**: docstrings are now enforced on `project/src/` — every
  module/class/function needs a docstring (`ruff`'s `D` rules, `pep257` convention), and every function
  parameter needs an actual reST `:param:` description, not just a type (`pydoclint`, `style=sphinx`).
  Root cause of the user's "I think it disappeared" observation: `ruff`'s own `D417` rule (missing
  argument descriptions) only recognizes Google/NumPy-style docstring sections — it silently never fires
  on Sphinx-style `:param:` field lists, confirmed by direct testing, so `ruff` alone could never have
  enforced this regardless of configuration; `pydoclint` was added specifically to close that gap.
  `tests/` is exempt (self-descriptive by name) and dataclass per-attribute checking is off
  (`check-class-attributes = false` — dataclasses have no explicit `__init__` in source for content-
  matching to key off; class-level docstrings are still required). Retrofitted `src/deontic_reasoner/{__init__,models}.py`
  and `scripts/check_file_coverage.py` to comply; added to both the local pre-commit hook and
  `ci.yml`. Separately discovered and disclosed to the user: `.git/hooks/` is never tracked by git
  (structurally excluded, not a `.gitignore` matter) — the pre-commit hook has been local-only on this
  machine since `/setup`; this fix (and any future hook edit) needs manual reapplication on any other
  clone. Not restructured into a tracked wrapper script — out of scope of what was asked, flagged here
  in case it's wanted later.
- User confirmed the local-hooks-untracked behavior is expected (not something to fix), and asked for a
  companion `pre-push` hook mirroring CI, plus the same for future `/setup` runs. Added
  `project/.git/hooks/pre-push` — runs all five gates unconditionally (ruff, radon, pydoclint,
  pytest+coverage, per-file coverage), matching `ci.yml` exactly, before any push reaches origin.
  Updated `.claude/commands/setup.md` (the reusable skill definition) step 11 to describe both hooks and
  step 5 to install/configure `pydoclint` by default — so a future `/setup` run on a new project gets
  this same rigor without re-deriving it. Verified the new hook passes by running it directly.
- **Settled**: dataclass field-level docstrings keep using `:param:` (user's explicit preference,
  confirmed after seeing that `:ivar:`/inline are the only two forms `pydoclint`'s sphinx-attribute
  parser recognizes for strict checking — verified by direct testing under every relevant setting
  combination, not a config mistake). Trade-off accepted: `[tool.pydoclint] check-class-attributes`
  stays `false` — `:param:`-documented attributes aren't machine-checked for per-field completeness,
  only the class-level docstring's existence is (`ruff` D101). No code change needed: `models.py`
  already used `:param:` in its class docstrings.
- **Branching correction**: the docstring-enforcement infra (pyproject.toml config, `pydoclint`
  dependency, `ci.yml` step, `__init__.py`/`check_file_coverage.py` docstrings) had been committed
  directly onto `story/1-core-data-model/1-propositional-core-types` — wrong, since it's project-wide
  tooling, not story-1-specific work, and doesn't belong bundled into that story's own PR. Corrected
  without rewriting any existing history (no force-push): recreated the same infra changes fresh on a
  new `chore/docstring-enforcement` branch off `main` (models.py doesn't exist on `main` yet, so the
  earlier commit couldn't be cherry-picked as-is), opened
  [PR #2](https://github.com/stefano-bragaglia/deontic-reasoner/pull/2) against `main` directly — the
  user will approve/merge it. On the story branch, `git revert`ed the misplaced commit (forward-only,
  PR #1 is back to just `models.py` + `test_models_core.py`, verified via `gh pr diff 1 --name-only`).
  **Follow-up once PR #2 merges**: fast-forward `feature/1-core-data-model` to the new `main`, merge it
  into the story branch, then re-add docstrings to `Rule`/`HardConstraint` in `models.py` as a fresh,
  properly-scoped commit (the content was reverted along with the infra, but the classes will need it
  again once the merged-in `pyproject.toml` starts requiring it).
- **Further correction**: `__init__.py`'s docstring itself was still wrongly on the infra branch — the
  user pointed out it's genuine package content belonging to whichever story establishes the package
  (story 1), not tooling. Fixing that surfaced a real mechanical tension: `ruff`'s `D104` requires a
  package docstring on *any* `__init__.py`, so the infra branch couldn't leave it untouched and still
  pass its own CI. Resolved by exempting `__init__.py` from `D104` entirely
  (`"**/__init__.py" = ["D104"]`) — package-level docstrings are now optional, entirely a story's own
  choice, never infra-mandated. PR #2 now touches zero `src/` files
  (`ci.yml`/`pyproject.toml`/`check_file_coverage.py`/`uv.lock` only); the package docstring was
  re-added on the story branch as its own commit. Both PRs' scopes verified via
  `gh pr diff <n> --name-only` — PR #1: `__init__.py`, `models.py`, `test_models_core.py`; PR #2: no
  `src/` files at all. CI green on both, on GitHub.
- **PR #2 merged** by the user. Ran the planned follow-up: fast-forwarded `feature/1-core-data-model` to
  the new `main` (pushed), merged it into the story branch (clean, no conflicts), then re-added reST
  docstrings to `Rule`/`HardConstraint` in `models.py` as their own commit — now required by the
  merged-in `pyproject.toml`. All gates (ruff, radon, pydoclint, pytest+coverage) pass locally and in
  GitHub Actions on PR #1. PR #1 was previously blocked on review specifically because it lacked these
  docstrings; that's now resolved.
- **PR #1 merged** by the user. On-merge steps done: pulled the merge into `feature/1-core-data-model`
  locally (fast-forward), deleted `story/1-core-data-model/1-propositional-core-types` (remote +
  local) and the now-merged `chore/docstring-enforcement` (remote + local, stale cleanup). Story
  `1-propositional-core-types` marked `merged`; its doc file renamed
  `1-propositional-core-types.md` → `1-DONE-propositional-core-types.md` with `DONE - ` inserted in
  its title. Feature `1-core-data-model` stays `branched` — stories 2 and 3 (`2-hohfeldian-norm-and-relation`,
  `3-json-round-trip`) are still only `approved`, not started.
- Stories 2 and 3 completed (Stage A → Stage B → PR → merge each), same pattern as story 1. All three
  merged; feature `1-core-data-model` set `stories-merged`, then epic PR
  [#5](https://github.com/stefano-bragaglia/deontic-reasoner/pull/5) opened against `main`. Auto-merge
  path per the branching model: `mergeable=MERGEABLE` (no conflict), required `test` CI check passed,
  merged via `gh pr merge --merge` — no separate human review needed at this tier, since it only
  replays already-individually-reviewed story diffs. Feature `1-core-data-model` is now `done`; epic
  file marked `DONE` (`0-core-data-model.md` → `0-DONE-core-data-model.md`). **First feature of this
  iteration fully shipped**: `Atom`/`World`/`Rule`/`HardConstraint`/`Relation`/`Norm` +
  `to_dict`/`from_dict`/`world_to_list`/`world_from_list`, all on `main`.
- New `ruff`/`pep257` rules hit for the first time in `preference.py` (first module with plain function,
  not class, docstrings): D400 (first line must end with a period) and D401 (first line must be
  imperative mood — "Check whether..." not "Does..."/"Is..."). Both only apply to function/method
  docstrings, not class docstrings, which is why `models.py`/`serialization.py`'s noun-phrase class
  docstrings never tripped this. Worth writing function docstrings in imperative mood from the start in
  later stories to avoid the same fix-up cycle.
- Story 2's acceptance criterion 7 (stable, sort-key-based weight summation, not raw hash-dependent
  iteration order) can't be directly exercised by a black-box unit test — a single test process has a
  fixed hash seed, so a `frozenset`'s iteration order never actually varies within one run regardless of
  whether the implementation sorts or not. Tested via the practical proxy that's actually achievable:
  `preferred(...)`'s result must be identical regardless of the order the caller's `rules` *list*
  argument is given in. This doesn't prove cross-process hash-seed independence directly, but it does
  prove the implementation doesn't leak list-argument order into the result, which is the property a
  caller can actually observe and rely on.

## Next Action
<!-- One sentence. What should happen next, and who does it (agent or user). -->
Run /stage-b 2-preference-ordering-and-best-worlds/3-best-worlds-with-scoped-enumeration (tests written;
last story of this feature).