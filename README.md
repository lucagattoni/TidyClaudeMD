# TidyClaudeMD

Claude Code reads several kinds of instruction files before it does anything else: your project's `CLAUDE.md`, custom `.claude/rules/`, `SKILL.md` procedures, and its own memory notes. As a project matures, these files accumulate — rules get duplicated across files, drift out of date, or grow past the length Claude actually reads reliably.

TidyClaudeMD is a Claude Code plugin that audits and slims these instruction files — never your actual code. It works the way you'd refactor, not just delete: content moves to its correct home, verbose rules get compressed, only confirmed-dead content is removed, and anything only you can judge is raised as a question instead of guessed at. A second skill learns from real tidy runs and improves the first — evidence-only, never a guess.

**Version 0.20.1** · [Changelog](CHANGELOG.md)

## New to Claude Code?

TidyClaudeMD assumes you already know what these files are. If you don't, here's the short version — full primer with links to Anthropic's own docs: [docs/claude-code-concepts.md](docs/claude-code-concepts.md).

| File | What it is |
|---|---|
| `CLAUDE.md` | Instructions you write; loaded into every session automatically |
| `.claude/rules/*.md` | CLAUDE.md split into topic files, optionally loaded only for matching file paths |
| `SKILL.md` (a Skill) | A packaged procedure Claude loads on demand, not every session |
| `MEMORY.md` (auto memory) | Notes Claude writes about *itself* — learnings from past sessions, not instructions from you |

## What TidyClaudeMD does

Two skills, bundled as one plugin:

| Skill | Purpose |
|---|---|
| [`/tidyclaudemd:claudemd-tidy`](skills/claudemd-tidy/SKILL.md) | Audits one of the file types above and slims it: relocate misplaced content, compress verbose content, delete only confirmed-dead content, or raise a question when only you can decide |
| [`/tidyclaudemd:claudemd-tidy-reflect`](skills/claudemd-tidy-reflect/SKILL.md) | Learns from real tidy runs and improves the tidy skill itself — evidence-only, never guesses |

Every tidy run: builds a picture of your repo → questions every line against seven tests → proposes verdicts → applies them (autonomously if the file is git-tracked and every change is revertible, otherwise with your confirmation first) → writes a permanent log of what happened. Full mechanics, every test and verdict, and the six safety invariants that self-improvement can never weaken: **[docs/reference.md](docs/reference.md)**.

## Install

`/tidyclaudemd:claudemd-tidy` reads its rules from a `## CLAUDE.md hygiene — keep every CLAUDE.md slim` section in your **global** `~/.claude/CLAUDE.md`. **If that section doesn't exist yet, the skill bootstraps it automatically** the first time you run it: it appends [`HYGIENE-RULES-TEMPLATE.md`](HYGIENE-RULES-TEMPLATE.md) verbatim to your global `CLAUDE.md` and tells you it did so — a one-time seed, not an ongoing copy.

```
/plugin marketplace add lucagattoni/TidyClaudeMD
/plugin install tidyclaudemd@TidyClaudeMD
```

Both skills are then available under their namespaced form shown in the table above. This is the only supported install path — no manual copy, no standalone script (see [`plans/plugin-packaging-plan.md`](plans/plugin-packaging-plan.md) for the full design).

Verified locally via `claude --plugin-dir` (skill loading, `--report` mode, and path resolution all confirmed working). **Not yet verified end-to-end via the real marketplace flow** (`/plugin marketplace add` + `/plugin install` from a genuinely separate checkout) — see the plan's phased action items for what's still open.

## File layout

This repo (also the plugin, and its own marketplace):

```
TidyClaudeMD/
  README.md                          ← this file
  docs/                              ← detailed reference, linked from here
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

Installed, this same content lives under `${CLAUDE_PLUGIN_ROOT}` (the plugin's installation directory — substituted inline wherever a skill's own instructions reference it, no hardcoded path). Persistent bookkeeping lives separately, under `${CLAUDE_PLUGIN_DATA}`, so it survives a plugin update instead of being replaced along with the bundled files:

```
${CLAUDE_PLUGIN_DATA}/
  RUNS-archive/      ← one permanent log file per run, `<date>-<time>-<target>.log`
                         (e.g. `20260708-2327-job2026.log`) — never deleted, never batched
  memory-backups/    ← full pre-edit snapshots, taken before any --memory run touches auto memory
```

Format of each run log: [docs/reference.md § Run logs](docs/reference.md#run-logs).

## Versioning

One semver for the whole suite, kept identical in both skills' frontmatter, this README's header, and `.claude-plugin/plugin.json`, with history in [CHANGELOG.md](CHANGELOG.md):

| Bump | When |
|---|---|
| MAJOR | Changed workflow contract — phases, stop points, file layout (always needs user approval) |
| MINOR | New step, verdict, signal, or capability |
| PATCH | Clarified wording, tightened an existing test, format fix |

## License

MIT — see [LICENSE](LICENSE).
