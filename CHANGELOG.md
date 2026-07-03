# Changelog — claude-md-tidy suite

All notable changes to the suite (`claude-md-tidy` + `claude-md-tidy-reflect`). Follows semantic versioning; one version for the whole suite. Entries produced by the self-improvement loop cite the run record that motivated them as `(run: YYYY-MM-DD <repo>)`.

## [Unreleased]

## [0.3.1] — 2026-07-03

_Version counting restarted at 0.1 on 2026-07-03: the suite is not yet considered stable/public-ready, so the history below was renumbered from the original 1.0.0–1.2.1 down to 0.1.0–0.3.1, preserving the original bump sizes and dates exactly. No behavior changed by the renumbering itself._

### Fixed
- DELETE and CHALLENGE verdicts overlapped on dead references (both claimed "references something that no longer exists"), so two runs could classify the same line differently. DELETE now requires a grep-*confirmed* dead reference or duplicate (cited evidence); anything the grep can't settle defaults to CHALLENGE, never a guessed DELETE (critical self-review, 2026-07-03).

## [0.3.0] — 2026-07-03

### Added
- Repo-picture step with a comprehension gate: the tidy skill must fully survey the repo (tree, docs, skills, hooks, settings, manifests, CI, git history) before analyzing any CLAUDE.md line, so verdicts are grounded in what the repo actually is (user feedback, 2026-07-03).
- Line-level critical interrogation (Step 2b): every line tested for True / Live / Consistent / Actionable against the repo picture (user feedback, 2026-07-03).
- Fifth verdict **CHALLENGE**: suspect lines are raised to the user as concrete questions with evidence and options — never resolved unilaterally; CHALLENGE items lead the plan (user feedback, 2026-07-03).
- Generalizability tagging of user feedback in run records (`[general → suggested home]` / `[repo-specific]` / `[general?]`), assessed at recording time; the reflect skill verifies the tags and treats `[general]` feedback as its top-priority signal (user feedback, 2026-07-03).

## [0.2.0] — 2026-07-03

### Added
- `claude-md-tidy-reflect` skill: evidence-only self-improvement loop that learns from run records and concrete CLAUDE.md instances, routes lessons to their correct home, and applies minimal diffs behind an invariant gate.
- Run recording (Step 7) in `claude-md-tidy`: every run appends a structured record to `RUNS.md` (result, user amendments, questions, friction, uncovered cases) — the training data for reflection.
- `README.md` documenting every feature, the file layout, the six protected invariants, and the versioning policy.
- This `CHANGELOG.md` and suite versioning (`version:` in both skills' frontmatter).

## [0.1.0] — 2026-07-03

### Added
- Initial `claude-md-tidy` skill: preflight (git sync + repo-visibility PRIMARY CHECK), repo-wide CLAUDE.md scan, four-verdict analysis (KEEP / COMPRESS / RELOCATE / DELETE) with evidence requirements, stop-and-confirm plan, disciplined apply (repo branching conventions, two-way cross-links, docs-in-sync grep, no-loss check), and final report.
- Rules single-sourced from the "CLAUDE.md hygiene" section of `~/.claude/CLAUDE.md`.
