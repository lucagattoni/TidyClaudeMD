# Changelog — TidyClaudeMD

All notable changes to the suite (`claudemd-tidy` + `claudemd-tidy-reflect`). Follows semantic versioning; one version for the whole suite. Entries produced by the self-improvement loop cite the run record that motivated them as `(run: YYYY-MM-DD <repo>)`.

## [Unreleased]

## [0.14.0] — 2026-07-08

### Added
- **Target-class registry** with the first new class, **user-level** (`--user`): `~/.claude/CLAUDE.md` and `~/.claude/rules/*.md` become first-class tidy targets, running the same phases with inverted Correctly-scoped? direction ("genuinely universal, or belongs to one project?") and managed-policy as their earlier-loaded scope. `--all` runs every class. For user-level runs, "the repo" is `~/.claude` itself — Step 0.6's versioning check decides its tier (mission plan Phase 2; user decision 2026-07-08).

## [0.13.0] — 2026-07-08

### Added
- **`~/.claude` versioning bootstrap** (Step 0 preflight): when a run may write under `~/.claude`, the skill checks whether it's a git repository and, if not, proposes `git init` with an ignore-all + allowlist `.gitignore` (shown in full — it's the safety mechanism; credentials/transcripts/memory/plugins stay untracked by default). Never auto-created — home-directory repo-init is a user-only decision; declining keeps `~/.claude` writes confirm-first. Any future remote must be verified private before push; memory is never tracked (mission plan Phase 1; user decision 2026-07-08: in-place allowlist repo over dotfiles-manager/mirror/none).

## [0.12.0] — 2026-07-08

### Added
- **`.claude/rules/` awareness** (read side): Step 2's orientation pass lists rules files recursively, distinguishing path-scoped (`paths:` frontmatter) from unconditional; unconditional rules are read in full for Step 2b's Consistent? check (they load *after* the project CLAUDE.md, so overlap is consistency/duplication, never Redundant-by-order). Step 1 now reads all earlier-loaded machine scopes: `~/.claude/rules/` and a managed-policy CLAUDE.md when present (plan: `plans/claude-md-scope-tensions-plan.md` item 5 part a; user directive 2026-07-08).
- **`.claude/rules/` as a RELOCATE destination** (write side): path-scoped rule files join the relocation-home inventory and the RELOCATE test — a fact/constraint mattering only for part of the codebase relocates to a path-scoped rule, a procedure still goes to a skill; never introduced into a repo that doesn't already use `.claude/rules/` without asking. New evidence rule: verbatim duplicates in an unconditional rule are DELETE-eligible, in a path-scoped rule CHALLENGE-only (deleting would narrow always-loaded content to conditional — a user-only scope decision) (plan item 5 part b).

### Changed
- RELOCATE test now cites the official every-session vs. sometimes/path-scoped framing directly, sharpening borderline COMPRESS-vs-RELOCATE calls (plan item 1, folded into the same table-row edit).

## [0.11.0] — 2026-07-08

### Added
- **`@import` detection and following** (general mechanism; AGENTS.md is one instance): Step 2 detects every live import (outside backticks/fenced code blocks — a code-span mention is never an import) and Step 2b interrogates the **effective** CLAUDE.md including imported lines (max four documented hops; no assumed cycle safety). External imports (`@~/...`, absolute outside paths) are hedged as possibly inert (declinable one-time approval dialog — a file-content-invisible state); in-repo imports are assumed live (not approval-gated) (plan: `plans/claude-md-scope-tensions-plan.md` item 4 parts b+c; user directive 2026-07-08).
- **AGENTS.md visibility check**: Step 2 reports whether a found AGENTS.md is imported by any CLAUDE.md (import or symlink — the only direction that matters), escalating to a repo-level CHALLENGE on unintentional-looking drift; also added to `--report` mode's mechanical checks. Never auto-adds an import (plan item 4 part a).
- **Imported-file edit policy** (Step 5): in-repo instruction files edited directly (Step 0 encryption/CI flags extended to them); dual-purpose files (`@README`, `@package.json`) never edited — findings land on the import line; out-of-repo files audited but CHALLENGE-only with individual confirmation (user decision 2026-07-08) (plan item 4 part b).

## [0.10.0] — 2026-07-08

### Added
- Sixth Step 2b question **Correctly-scoped?**: flags a project CLAUDE.md line that is really a generic cross-project preference (→ user `~/.claude/CLAUDE.md`) or a personal repo-specific preference (→ `CLAUDE.local.md`); fires only when the content isn't already in an earlier-loaded scope (that's Redundant-by-order's case). Routes to CHALLENGE only — scope intent is a user-only decision. New Step 4 resolutions *promote-to-global* / *move-to-local*, with the new rule that **out-of-repo writes are confirmed individually, never bundled into a bulk approve-all** (apply-phase discipline chosen over a seventh invariant — user decision 2026-07-08) (plan: `plans/claude-md-scope-tensions-plan.md` item 2; user directive 2026-07-08).
- Seventh Step 2b question **Rule-vs-memory?**: flags a line that reads like an agent-discovered pattern rather than a deliberate team rule. CHALLENGE-only; the *belongs-in-memory* resolution writes nothing and removes nothing — the user moves it manually, and only a later run with a targeted grep confirming the content in memory may DELETE the original (no-loss preserved) (plan item 3; user directive 2026-07-08).

## [0.9.1] — 2026-07-03

### Fixed
- `.claude-plugin/marketplace.json` had an invalid schema, caught when the user actually ran `/plugin marketplace add lucagattoni/TidyClaudeMD` for real and it failed validation: missing the required top-level `name` (string) and `owner` (object) fields, and using an invented `source: {type, url, path}` object plus a `latest` field, neither of which exist in the real schema. Fixed against `claude-plugins-official`'s own working marketplace.json (installed locally at `~/.claude/plugins/marketplaces/claude-plugins-official/`): a same-repo plugin's `source` is just the relative-path string `"."`. Corrected the manifest itself, `claudemd-tidy-reflect` Step 5 sub-step 7 (no `marketplace.json` field to update on version bumps — only `plugin.json`'s `version` matters), and every doc reference to the now-removed `latest` field (user report, 2026-07-03).

## [0.9.0] — 2026-07-03

### Added
- `HYGIENE-RULES-TEMPLATE.md`: the plugin now bundles a static seed template of the `## CLAUDE.md hygiene` section. `claudemd-tidy` Step 1 bootstraps it automatically into `~/.claude/CLAUDE.md` the first time the section is found missing, instead of stopping and reporting — resolves the gap documented in v0.8.1 all the way to "just works" instead of "clearly explained but still manual" (user request, 2026-07-03).

### Changed
- Invariant 5 reworded from "the skills never carry a private copy" to "single-sourced at runtime" — the plugin does now carry a one-time bootstrap copy by design; the invariant that matters (no *ongoing* parallel copy read at runtime, no drift from the global file after first install) is unchanged. Touches a protected invariant; applied only after presenting the tradeoff and getting explicit approval (two decisions: static-seed vs. auto-synced template, and skill-level bootstrap vs. a `SessionStart` hook — user chose static seed + skill-level check both times, 2026-07-03).

### Known limitation
- The bootstrap path itself (Step 1 appending the template when the section is missing) is **not independently tested** — safely sandboxing a test requires either overriding `HOME` (which breaks Claude Code auth) or selectively copying credential files into a fake home (correctly blocked by this environment's own safety classifier as credential exploration). The logic is simple, unambiguous prompt instruction using the same Write/Edit mechanism already validated working in the v0.8.0 `--report`-mode test (which successfully located and attempted a write to `${CLAUDE_PLUGIN_DATA}/RUNS.md`), but the specific append-if-missing behavior has not been run end to end.

## [0.8.1] — 2026-07-03

### Added
- README Install section gained a documented prerequisite: the `## CLAUDE.md hygiene` section in the user's global `~/.claude/CLAUDE.md` is a hard dependency for `/tidyclaudemd:claudemd-tidy` regardless of install method, and was previously undocumented — a fresh plugin install with no prior manual-install history would have installed successfully then failed on first use with no explanation (user feedback, 2026-07-03).

## [0.8.0] — 2026-07-03

_Executes `plans/plugin-packaging-plan.md` phases 1-5 and 7 in one release (user request, 2026-07-03): the rename, the path-portability fix, the manifest files, the reflect skill's versioning-integration sub-step, a local `--plugin-dir` test, and this README rewrite. Phase 6 (external marketplace-flow verification from a genuinely separate machine/checkout) is explicitly not done — it isn't verifiable from within the session/machine that authored the plugin._

### Verified (local test, `claude --plugin-dir`)
- `/tidyclaudemd:claudemd-tidy` loads and triggers correctly under its namespaced form.
- `--report` mode runs preflight (encryption/CI/visibility checks) and produces the designed two-tier output (mechanical checks + explicitly-labeled-unverified verdict impression) against a scratch repo.
- `${CLAUDE_PLUGIN_DATA}` resolves to a real path outside the target project (`~/.claude/plugins/data/tidyclaudemd-inline/RUNS.md` for a `--plugin-dir`-loaded plugin), confirming the path-portability fix. The actual write was blocked by this test session's own sandbox restrictions on cross-directory writes in non-interactive mode (not a plugin defect) — the resolved *path* is confirmed, the write itself is not.

### Changed
- **Both skills renamed**, live: `claude-md-tidy` → `claudemd-tidy`, `claude-md-tidy-reflect` → `claudemd-tidy-reflect`. Directories, both `SKILL.md` frontmatter `name:` fields, every cross-reference between the two skills, the README, and this CHANGELOG's historical entries were all updated to match (user decision, 2026-07-03).
- **All hardcoded `~/.claude/skills/claude-md-tidy/…` paths replaced** with Claude Code's plugin environment variables: `${CLAUDE_PLUGIN_DATA}` for `RUNS.md`/`RUNS-archive/` (persists across plugin updates), `${CLAUDE_PLUGIN_ROOT}` for the read-only cross-references to `SKILL.md`/`README.md`/`CHANGELOG.md` (the current version's bundled files). This was the one real blocker the plugin plan had flagged — a plugin published with the old hardcoded paths would have silently been unable to find its own run records (`plans/plugin-packaging-plan.md`).
- Invocation is now namespaced throughout every doc: `/tidyclaudemd:claudemd-tidy` and `/tidyclaudemd:claudemd-tidy-reflect`, not the bare pre-plugin form.
- `claudemd-tidy-reflect` Step 5 gained a seventh sub-step: keep `.claude-plugin/plugin.json`'s `version` and `.claude-plugin/marketplace.json`'s `latest` field in sync with every future version bump.

### Added
- `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`: this repo is now both the Claude Code plugin (`tidyclaudemd`, bundling both skills) and its own marketplace listing.

**Bump note:** renaming live directories and adding new root-level manifest files is arguably a MAJOR "file layout" change per this suite's own versioning table. Consistent with the same call made for `RUNS-archive/` in v0.4.0 (item 11), this is applied as MINOR instead — the project is still pre-1.0 and under active development; jumping to 1.0.0 for an unverified-end-to-end plugin release would misrepresent stability, not reflect it.

## [0.7.3] — 2026-07-03

### Added
- `LICENSE` (MIT) at repo root (user decision, 2026-07-03).
- README `## License` section pointing to it.

### Changed
- Plugin packaging plan (`plans/plugin-packaging-plan.md`) revised per three user decisions: both skills are renamed `claudemd-tidy` / `claudemd-tidy-reflect` as part of the plugin work (not yet executed against the live repo); distribution is plugin-only going forward — no manual-install fallback, no `install.sh`; license is MIT. Also confirmed the exact runtime path-resolution mechanism (`${CLAUDE_PLUGIN_ROOT}` / `${CLAUDE_PLUGIN_DATA}`) that an earlier draft had left as an open question (user decisions + follow-up doc research, 2026-07-03).
- README's Install section reframed: manual copy is now explicitly a stopgap until the plugin ships, not a documented long-term alternative; the install.sh idea is dropped.

## [0.7.2] — 2026-07-03

### Added
- README `## Install` section documenting the current manual-copy install method, plus a `plans/plugin-packaging-plan.md` planning doc for two future distribution paths: a one-line `install.sh` and installation as a proper Claude Code plugin (user request, 2026-07-03).
- A prominent CHANGELOG.md link next to the version number at the top of the README, not just in the Versioning section at the bottom (user request, 2026-07-03).

## [0.7.1] — 2026-07-03

### Fixed
- Project branding left over from before the repo rename: README.md's title, its file-layout diagram's root directory name, CHANGELOG.md's title, and the reflect skill's frontmatter description all still said "claude-md-tidy suite" / `claudemd-tidy` (an unrelated old placeholder-directory typo, distinct from the skill-name rename below) instead of `TidyClaudeMD` (user feedback, 2026-07-03).

## [0.7.0] — 2026-07-03

### Added
- Fifth line-interrogation question **Redundant-by-order?**: does an earlier-loaded scope (per Claude Code's config load order) already establish a line's intent regardless of exact wording? Requires Step 1 to read the whole global CLAUDE.md, not just the hygiene section. Routes to COMPRESS or CHALLENGE, never DELETE — same-intent-different-wording redundancy isn't a verbatim duplicate or a grep-confirmed dead reference, so widening DELETE's evidence rule to cover it would weaken protected invariant 3 (competitive landscape review, 2026-07-03; `plans/v1.3.0-integrated-candidate-improvements.md` item 13).
- Target-finding (Step 2) now also globs the filesystem directly for gitignored personal-override filenames (`CLAUDE.local.md`-style), which `git ls-files` and a standard untracked-file check both miss entirely (competitive landscape review, 2026-07-03; item 14).
- Documented an optional, advisory `PostToolUse` hook pattern (README) that reminds a user to run `/claudemd-tidy` when a CLAUDE.md crosses the hygiene guardrail between runs — opt-in, not shipped as part of either skill (competitive landscape review, 2026-07-03; item 12).
- `claudemd-tidy-reflect` Step 1 gained a fourth evidence source: an ad-hoc lesson raised directly in an ordinary conversation, not tied to any recorded run — the conversation stands in as the citation (critical self-review, 2026-07-03; item 6).

### Changed
- RELOCATE's pointer requirement changed from "a one-line pointer" to a **rich-abstract pointer** — a short synopsis of the concrete fact(s) a reader would otherwise open the sub-doc for, plus the link — since a bare cross-reference gets opened anyway on any non-trivial task (competitive landscape review, 2026-07-03; item 10).
- CHALLENGE items in Step 4's plan are now grouped by stakes (destructive/safety-relevant first, then widely-depended-on rules, then minor CHALLENGEs batched last) instead of presented as a flat list (critical self-review, 2026-07-03; item 5).
- Step 2b's Consistent? question now actively probes common contradiction shapes (autonomy vs. ask-first, default vs. undocumented exception, conflicting thresholds, "always X" vs. a not-X case) instead of relying only on whatever a line-by-line read happens to surface (competitive landscape review, 2026-07-03; item 5).
- `claudemd-tidy-reflect` Step 5's README-sync rule tightened from "if any documented feature changed" to "check the README bullet for every step touched this pass" (critical self-review, 2026-07-03; item 4).

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
- Pruning signal: an instruction in `claudemd-tidy/SKILL.md` that never triggered across processed run records (never cited as the reason a CHALLENGE/DELETE/RELOCATE fired) is now a removal candidate, evidenced the same way an addition is; requires a Step 7 run-record format extension ("Instructions exercised") to be evaluable (competitive landscape review, 2026-07-03; `plans/v1.3.0-integrated-candidate-improvements.md` item 1).
- Provisional lessons: a lesson drawn from a single run record is now marked `provisional` in both the CHANGELOG and the source `RUNS.md` record; a later run's contradicting evidence demotes/revisits it, a second independent corroboration promotes it (critical self-review, 2026-07-03; item 3).
- `RUNS.md` archiving: fully-processed, non-provisional, unpinned records beyond the 3 most recent are moved (never deleted) into a new `RUNS-archive/` directory, keeping the always-loaded `RUNS.md` bounded. A `Pinned: yes` record marker exempts a record from archiving indefinitely (competitive landscape review, 2026-07-03; item 11 — applied as MINOR per explicit user decision, since it is purely additive; the suite's own file-layout-is-MAJOR reading was noted but not applied this round).

### Changed
- Run-record format (`claudemd-tidy` Step 7) gained an "Instructions exercised" field tracing which step/test produced each non-KEEP verdict and each CHALLENGE (prerequisite for the pruning signal above).

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
- `claudemd-tidy-reflect` skill: evidence-only self-improvement loop that learns from run records and concrete CLAUDE.md instances, routes lessons to their correct home, and applies minimal diffs behind an invariant gate.
- Run recording (Step 7) in `claudemd-tidy`: every run appends a structured record to `RUNS.md` (result, user amendments, questions, friction, uncovered cases) — the training data for reflection.
- `README.md` documenting every feature, the file layout, the six protected invariants, and the versioning policy.
- This `CHANGELOG.md` and suite versioning (`version:` in both skills' frontmatter).

## [0.1.0] — 2026-07-03

### Added
- Initial `claudemd-tidy` skill: preflight (git sync + repo-visibility PRIMARY CHECK), repo-wide CLAUDE.md scan, four-verdict analysis (KEEP / COMPRESS / RELOCATE / DELETE) with evidence requirements, stop-and-confirm plan, disciplined apply (repo branching conventions, two-way cross-links, docs-in-sync grep, no-loss check), and final report.
- Rules single-sourced from the "CLAUDE.md hygiene" section of `~/.claude/CLAUDE.md`.
