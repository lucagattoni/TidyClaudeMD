# Changelog — TidyClaudeMD

All notable changes to the suite (`claude-md-tidy` + `claude-md-tidy-reflect`). Follows semantic versioning; one version for the whole suite. Entries produced by the self-improvement loop cite the run record that motivated them as `(run: YYYY-MM-DD <repo>)`.

## [Unreleased]

## [0.7.2] — 2026-07-03

### Added
- README `## Install` section documenting the current manual-copy install method, plus a `plans/plugin-packaging-plan.md` planning doc for two future distribution paths: a one-line `install.sh` and installation as a proper Claude Code plugin (user request, 2026-07-03).
- A prominent CHANGELOG.md link next to the version number at the top of the README, not just in the Versioning section at the bottom (user request, 2026-07-03).

## [0.7.1] — 2026-07-03

### Fixed
- Project branding left over from before the repo rename: README.md's title, its file-layout diagram's root directory name, CHANGELOG.md's title, and the reflect skill's frontmatter description all still said "claude-md-tidy suite" / `claudemd-tidy` instead of `TidyClaudeMD` (user feedback, 2026-07-03).

## [0.7.0] — 2026-07-03

### Added
- Fifth line-interrogation question **Redundant-by-order?**: does an earlier-loaded scope (per Claude Code's config load order) already establish a line's intent regardless of exact wording? Requires Step 1 to read the whole global CLAUDE.md, not just the hygiene section. Routes to COMPRESS or CHALLENGE, never DELETE — same-intent-different-wording redundancy isn't a verbatim duplicate or a grep-confirmed dead reference, so widening DELETE's evidence rule to cover it would weaken protected invariant 3 (competitive landscape review, 2026-07-03; `plans/v1.3.0-integrated-candidate-improvements.md` item 13).
- Target-finding (Step 2) now also globs the filesystem directly for gitignored personal-override filenames (`CLAUDE.local.md`-style), which `git ls-files` and a standard untracked-file check both miss entirely (competitive landscape review, 2026-07-03; item 14).
- Documented an optional, advisory `PostToolUse` hook pattern (README) that reminds a user to run `/claude-md-tidy` when a CLAUDE.md crosses the hygiene guardrail between runs — opt-in, not shipped as part of either skill (competitive landscape review, 2026-07-03; item 12).
- `claude-md-tidy-reflect` Step 1 gained a fourth evidence source: an ad-hoc lesson raised directly in an ordinary conversation, not tied to any recorded run — the conversation stands in as the citation (critical self-review, 2026-07-03; item 6).

### Changed
- RELOCATE's pointer requirement changed from "a one-line pointer" to a **rich-abstract pointer** — a short synopsis of the concrete fact(s) a reader would otherwise open the sub-doc for, plus the link — since a bare cross-reference gets opened anyway on any non-trivial task (competitive landscape review, 2026-07-03; item 10).
- CHALLENGE items in Step 4's plan are now grouped by stakes (destructive/safety-relevant first, then widely-depended-on rules, then minor CHALLENGEs batched last) instead of presented as a flat list (critical self-review, 2026-07-03; item 5).
- Step 2b's Consistent? question now actively probes common contradiction shapes (autonomy vs. ask-first, default vs. undocumented exception, conflicting thresholds, "always X" vs. a not-X case) instead of relying only on whatever a line-by-line read happens to surface (competitive landscape review, 2026-07-03; item 5).
- `claude-md-tidy-reflect` Step 5's README-sync rule tightened from "if any documented feature changed" to "check the README bullet for every step touched this pass" (critical self-review, 2026-07-03; item 4).

## [0.6.0] — 2026-07-03

### Added
- Encryption check (Step 0 Preflight): detects git-crypt/SOPS filters or an `.age` key reference; any encryption-unlock instructions are force-classified KEEP, and any other RELOCATE destination must be confirmed within the same encryption scope before Step 5 executes it — otherwise the block routes to CHALLENGE (competitive landscape review, 2026-07-03; `plans/v1.3.0-integrated-candidate-improvements.md` item 8).
- CI-dependency check (Step 0 Preflight): detects CI scripts that reference `CLAUDE.md` by filename; flagged content is ineligible for RELOCATE without the user's explicit confirmation in Step 4, otherwise it routes to CHALLENGE (competitive landscape review, 2026-07-03; item 9).

## [0.5.0] — 2026-07-03

### Added
- Report mode (`--report`): a cheap, no-plan, no-edits pre-check — mechanical line/token counts vs. the hygiene guardrail, non-English/CJK token-overhead flags, a session-cost estimate, plus a shallow verdict-mix impression explicitly labeled as unverified judgment. Skips the survey and line interrogation entirely; still logs a Step 7 run record (competitive landscape review, 2026-07-03; `plans/v1.3.0-integrated-candidate-improvements.md` item 7).

### Changed
- Repo survey (Step 2) is now **claim-driven** instead of an unbounded full-repo read: a fixed cheap orientation pass (README, commands/skills listing, settings.json) plus targeted verification of only the concrete claims each CLAUDE.md actually makes. The comprehension gate's "if you can't, keep reading" is redefined as "keep resolving unverified claims" — cost now scales with the CLAUDE.md's own length, not the repo's size (competitive landscape review, 2026-07-03; item 2).

## [0.4.0] — 2026-07-03

### Added
- Pruning signal: an instruction in `claude-md-tidy/SKILL.md` that never triggered across processed run records (never cited as the reason a CHALLENGE/DELETE/RELOCATE fired) is now a removal candidate, evidenced the same way an addition is; requires a Step 7 run-record format extension ("Instructions exercised") to be evaluable (competitive landscape review, 2026-07-03; `plans/v1.3.0-integrated-candidate-improvements.md` item 1).
- Provisional lessons: a lesson drawn from a single run record is now marked `provisional` in both the CHANGELOG and the source `RUNS.md` record; a later run's contradicting evidence demotes/revisits it, a second independent corroboration promotes it (critical self-review, 2026-07-03; item 3).
- `RUNS.md` archiving: fully-processed, non-provisional, unpinned records beyond the 3 most recent are moved (never deleted) into a new `RUNS-archive/` directory, keeping the always-loaded `RUNS.md` bounded. A `Pinned: yes` record marker exempts a record from archiving indefinitely (competitive landscape review, 2026-07-03; item 11 — applied as MINOR per explicit user decision, since it is purely additive; the suite's own file-layout-is-MAJOR reading was noted but not applied this round).

### Changed
- Run-record format (`claude-md-tidy` Step 7) gained an "Instructions exercised" field tracing which step/test produced each non-KEEP verdict and each CHALLENGE (prerequisite for the pruning signal above).

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
