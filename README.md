# TidyClaudeMD

**Version 0.15.0** ([changelog](CHANGELOG.md)) · Two personal Claude Code skills that keep every repo's `CLAUDE.md` slim without losing information — and that improve themselves from the experience of real runs.

Distributed as a single Claude Code plugin, `tidyclaudemd`, bundling both skills. This repo is the plugin **and** its own marketplace — no separate repo to publish to.

| Skill | Purpose |
|---|---|
| [`/tidyclaudemd:claudemd-tidy`](skills/claudemd-tidy/SKILL.md) | Audit and slim the CLAUDE.md file(s) of the current repo |
| [`/tidyclaudemd:claudemd-tidy-reflect`](skills/claudemd-tidy-reflect/SKILL.md) | Learn from recorded runs and meta-improve the suite itself |

## Install

`/tidyclaudemd:claudemd-tidy` reads its rules from a `## CLAUDE.md hygiene — keep every CLAUDE.md slim` section in your **global** `~/.claude/CLAUDE.md`. **If that section doesn't exist yet, Step 1 bootstraps it automatically** the first time you run the skill: it appends [`HYGIENE-RULES-TEMPLATE.md`](HYGIENE-RULES-TEMPLATE.md) verbatim to your global `CLAUDE.md` and tells you it did so. This is a one-time seed, not an ongoing copy — from that point on your global `CLAUDE.md` is the only file ever read or edited; the bundled template is never consulted again.

```
/plugin marketplace add lucagattoni/TidyClaudeMD
/plugin install tidyclaudemd@TidyClaudeMD
```

Both skills are then available under their namespaced form shown in the table above. This is the only supported install path — no manual copy, no standalone script (see [`plans/plugin-packaging-plan.md`](plans/plugin-packaging-plan.md) for the full design, including the two Claude Code environment variables, `${CLAUDE_PLUGIN_ROOT}` and `${CLAUDE_PLUGIN_DATA}`, that make this work without hardcoding install paths).

Verified locally via `claude --plugin-dir` (skill loading, `--report` mode, and path resolution all confirmed working). **Not yet verified end-to-end via the real marketplace flow** (`/plugin marketplace add` + `/plugin install` from a genuinely separate checkout) — see the plan's phased action items for what's still open.

## File layout

This repo (also the plugin, and its own marketplace):

```
TidyClaudeMD/
  README.md                          ← this file
  CHANGELOG.md                       ← version history of the suite (semver)
  LICENSE                            ← MIT
  HYGIENE-RULES-TEMPLATE.md          ← one-time bootstrap seed for ~/.claude/CLAUDE.md (see Install)
  .claude-plugin/
    plugin.json                      ← plugin manifest (name, version, license, ...)
    marketplace.json                 ← marketplace catalog (this repo lists itself)
  skills/
    claudemd-tidy/SKILL.md           ← the tidy skill
    claudemd-tidy-reflect/SKILL.md   ← the self-improvement skill
```

Installed, this same content lives under `${CLAUDE_PLUGIN_ROOT}` (the plugin's installation directory — substituted inline wherever a skill's own instructions reference it, no hardcoded path). Persistent bookkeeping lives separately, under `${CLAUDE_PLUGIN_DATA}`, specifically so it survives a plugin update instead of being replaced along with the bundled files:

```
${CLAUDE_PLUGIN_DATA}/
  RUNS.md            ← run records (created on first run; newest at top) — accumulates real
                         per-repo feedback that can include private context
  RUNS-archive/      ← fully-processed, non-provisional, unpinned records moved out of RUNS.md
                         once it has more than 3 archive-eligible records — never deleted
```

## /tidyclaudemd:claudemd-tidy — features

- **Rule sourcing.** The skill carries no rules of its own: it reads the numbered rules from the "CLAUDE.md hygiene" section of `~/.claude/CLAUDE.md` at runtime. Editing that section changes the skill's behavior; if the section is missing, the skill bootstraps it from the bundled template (see Install) rather than stopping.
- **Scope.** Target classes: project CLAUDE.md files (default), user-level files (`--user`: your global `~/.claude/CLAUDE.md` and `~/.claude/rules/`), and skills (`--skills`: SKILL.md description quality, progressive disclosure, frontmatter sanity — never this suite's own two skills, which only the reflect loop may change) — same phases, class-specific tests; `--all` runs every class. By default it sweeps every CLAUDE.md in the repo, root and nested — including gitignored personal-override files (e.g. `CLAUDE.local.md`-style conventions), which a plain `git ls-files`/untracked-file check would otherwise miss entirely. The audit covers the **effective** CLAUDE.md: live `@imports` are detected (code-span mentions excluded) and followed up to the documented four-hop limit, so imported content — an `@AGENTS.md`, a `@docs/...` file — is interrogated like inline content. External (out-of-repo) imports are hedged as possibly inert, since a declined approval dialog silently disables them. An unimported `AGENTS.md` is flagged: Claude Code never reads it, in any session. Edits follow a strict policy: in-repo instruction files are edited in place, dual-purpose files (README, manifests) never — findings land on the import line — and out-of-repo files only via individually-confirmed CHALLENGEs.
- **Report mode (`--report`).** A cheap, no-plan, no-edits pre-check: mechanical line/token counts vs. the hygiene guardrail, non-English token-overhead flags, a session-cost estimate, and a shallow verdict-mix impression explicitly labeled as unverified judgment. Skips the survey and line interrogation entirely — not a substitute for a full tidy, but still logs a Step 7 run record.
- **Preflight.** Verifies it's in a git repo, syncs (`git fetch`, branch/dirty/ahead-behind check), and checks repo visibility — public or unknown visibility activates the global PRIMARY CHECK for everything the skill writes. Also checks for encryption (git-crypt/SOPS/age) and CI scripts that depend on `CLAUDE.md`'s content (or on any imported file's) — either flag routes the affected block to CHALLENGE instead of RELOCATE until the flag is resolved. When a run may write under `~/.claude`, preflight offers (never forces) a one-time versioning bootstrap: a git repo with an ignore-all + allowlist `.gitignore` so instruction files get history and reverts while credentials, transcripts, memory, and plugins stay untracked by default.
- **Repo picture first, claim-driven.** Before analyzing a single line, the skill runs a fixed, cheap orientation pass (README, a listing of commands/skills, settings.json, and `.claude/rules/` — unconditional rules read in full since they're always-loaded context; path-scoped ones noted) plus targeted verification of every concrete claim the CLAUDE.md makes (file paths, commands, workflows, tool references) — not an open-ended survey of the whole repo, so cost scales with the CLAUDE.md's own length. Must still pass a comprehension gate (what the repo does, how it runs, which agents operate on it, which conventions demonstrably hold) before analyzing any line; an unresolved gate means more claims to verify, never a wider survey. Every verdict is grounded in what the repo *is*, not in what the CLAUDE.md claims.
- **Line-level critical interrogation.** Every line is questioned against seven tests before any slimming verdict: **True?** (matches the repo picture), **Live?** (still governs something that exists), **Consistent?** (no contradiction with other rules, nested CLAUDE.md files, skills, or observable conventions — actively probed against common contradiction shapes: autonomy vs. ask-first, default vs. undocumented exception, conflicting thresholds, "always X" vs. a not-X case), **Actionable?** (two sessions would follow it the same way), **Redundant-by-order?** (does an earlier-loaded scope already establish this same intent, per Claude Code's config load order?), **Correctly-scoped?** (is this genuinely about this repo, or a cross-project preference belonging in the global CLAUDE.md / a personal preference belonging in CLAUDE.local.md?), **Rule-vs-memory?** (a deliberate team rule, or something that reads like an agent-discovered pattern?). Failing True/Live/Consistent/Actionable routes the line to CHALLENGE; failing only Redundant-by-order routes to COMPRESS or CHALLENGE, never straight to DELETE; failing only Correctly-scoped/Rule-vs-memory routes to CHALLENGE — scope and intent are user-only decisions. Scope resolutions can promote a line to the global CLAUDE.md or move it to CLAUDE.local.md; any write outside the repo is confirmed individually, never bundled in a bulk approval. A belongs-in-memory resolution never writes or deletes anything itself — the user moves the content, and a later run deletes the original only after verifying it landed.
- **Five-verdict analysis.** Every block of every CLAUDE.md gets exactly one verdict:
  - **KEEP** — behavior-changing rule, already terse.
  - **COMPRESS** — stays in place, rewritten tighter.
  - **RELOCATE** — detail moves to its natural owner (a skill/command, a path-scoped `.claude/rules/` file for facts that only matter in part of the codebase, `docs/`, README) with a **rich-abstract pointer** left behind: a short synopsis of the concrete fact(s) a reader would otherwise open the sub-doc for, not a bare "see docs/x.md" link.
  - **DELETE** — only for duplicated content (cited location) or a **grep-confirmed** dead reference (cited failed lookup, or the found new location if renamed).
  - **CHALLENGE** — suspect content the grep can't settle either way: ambiguous liveness, an unconfirmed possible rename, a contradiction, wording too vague to act on, an unresolved encryption/CI-dependency flag, or same-intent-different-wording redundancy with no confirmed added nuance. Never resolved unilaterally — always raised to the user with the conflicting evidence and options (fix / keep / delete, plus promote-to-global / move-to-local for scope items and belongs-in-memory for rule-vs-memory items). An inconclusive search defaults here, never to a guessed DELETE.
- **Stop point.** The full plan (CHALLENGE questions **grouped by stakes** — destructive/safety-relevant first, then widely-depended-on rules, then everything else batched as minor CHALLENGEs — then the verdict table, destinations, evidence, before/after line estimates) is presented and **nothing is edited until the user confirms**. Partial approval and amended verdicts are supported.
- **Apply discipline.** Follows the target repo's own branching conventions; relocates first (with two-way cross-links), then compresses, then deletes; greps for every moved term and updates all referencing docs in the same change; updates the repo's CHANGELOG if it keeps one.
- **No-loss check.** After applying, every removed line must be found at its new home or on the verified-DELETE list; the final CLAUDE.md is re-read end-to-end for coherence.
- **Run recording with generalizability tags.** Every run — even analyze-only — appends a structured record to `RUNS.md`: result, which instructions were exercised (which step/test produced each non-KEEP verdict and each CHALLENGE), user feedback, questions asked, friction, uncovered cases. Each piece of user feedback (amendments, CHALLENGE resolutions, remarks) is tagged **at recording time, while context is fresh**: `[general → suggested home]` if the lesson would hold in other repos, `[repo-specific]` if not, `[general?]` when unsure. These records are the training data for the reflection loop.

## /tidyclaudemd:claudemd-tidy-reflect — features

- **Evidence-only improvement.** Changes to the suite must trace to a concrete run record, the current session's live tidy experience, a dry-run analysis of a specific CLAUDE.md passed as an argument, or an ad-hoc lesson raised directly in an ordinary conversation (the conversation itself stands in as the citation). No evidence → no change, by design.
- **Signal extraction**, strongest first: user feedback pre-tagged `[general]`/`[general?]` in run records (verified, then treated as top-priority lessons; `[repo-specific]` items never enter the suite); user-overridden verdicts → the classification tests mis-sorted something real; questions the skill had to ask → instruction gaps; apply-phase friction → steps to harden; content outside the taxonomy → extend the verdicts; an instruction that never fired across processed records → a pruning candidate.
- **Three lesson tests.** Every lesson must be *generalizable* (helps other repos too), *traceable* (cites its run), and *correctly homed* — or it is discarded and reported as such.
- **Lesson routing.** Tidy mechanics → tidy SKILL.md; reflection mechanics → the reflect skill's own SKILL.md (the loop applies to itself); definitions of good CLAUDE.md content → the global hygiene rules (proposed to the user, never forked into the skill); repo-specific quirks → that repo's CLAUDE.md, never the general skill.
- **Provisional lessons.** A lesson drawn from a single run record is applied as **provisional** (marked in the CHANGELOG and on the source record); a later run that contradicts it demotes/revisits it, while a second independent corroboration promotes it. Keeps the loop from baking overfit or one-off feedback into a suite meant to work across arbitrary repos.
- **Pruning.** Reflection isn't only additive — an instruction that never triggered across processed run records is a removal candidate, evidenced the same way an addition is.
- **Bookkeeping.** Applies minimal diffs, bumps the suite version, writes the CHANGELOG entry with evidence citations, checks this README's bullet for every step touched this pass (paraphrased, never a verbatim restatement of a step's wording), marks consumed run records `Processed: yes (vX.Y.Z)` (or `, provisional`), and archives fully-processed, non-provisional, unpinned records into `RUNS-archive/` once `RUNS.md` holds more than the 3 most recent.

## Invariants (never weakened autonomously)

Self-improvement may not remove or weaken these; any lesson touching them requires explicit user approval with pros/cons:

1. **Stop-and-confirm** — no CLAUDE.md is edited before the user approves the plan.
2. **No-loss guarantee** — slimming relocates content; information is never lost.
3. **Evidence before DELETE** — every deletion cites a located duplicate or a verified-gone reference.
4. **PRIMARY CHECK** — public/unknown-visibility repos get the sensitive-content gate on every written file.
5. **Single-sourced rules at runtime** — every tidy run reads the hygiene rules only from `~/.claude/CLAUDE.md`; the skill never reads its own copy once installed. (The plugin does bundle a one-time bootstrap template, `HYGIENE-RULES-TEMPLATE.md`, used only to seed a fresh `~/.claude/CLAUDE.md` if the section is missing — after that first install, only the global file is ever read or edited. User decision, 2026-07-03.)
6. **No evidence, no change** — the reflect skill never improvises improvements.

## Optional companion hook

`/tidyclaudemd:claudemd-tidy` is judgment-driven and only runs when invoked, so nothing notices a CLAUDE.md crossing the global hygiene guardrail (~150 lines) *between* runs. This is deliberate — a deterministic hard cap would conflict with the five-test, judgment-driven verdicts above — but a purely **advisory** reminder closes the "drift happened" → "someone noticed" gap without blocking anything. Example `PostToolUse` hook (adapt the line-count check to your shell):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "f=\"$CLAUDE_TOOL_INPUT_FILE_PATH\"; case \"$f\" in *CLAUDE.md) n=$(wc -l < \"$f\"); if [ \"$n\" -gt 150 ]; then echo \"$f is now $n lines (> the ~150-line hygiene guardrail) — consider running /tidyclaudemd:claudemd-tidy\" >&2; fi ;; esac"
          }
        ]
      }
    ]
  }
}
```

This is guidance, not a shipped part of either skill — install it only if you want the reminder; nothing in the suite depends on it.

## Versioning

One semver for the whole suite, kept identical in both skills' frontmatter, this README's header, and `.claude-plugin/plugin.json`, with history in [CHANGELOG.md](CHANGELOG.md):

| Bump | When |
|---|---|
| MAJOR | Changed workflow contract — phases, stop points, file layout (always needs user approval) |
| MINOR | New step, verdict, signal, or capability |
| PATCH | Clarified wording, tightened an existing test, format fix |

## License

MIT — see [LICENSE](LICENSE).
