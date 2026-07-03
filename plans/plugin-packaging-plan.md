# Plugin packaging plan — installable Claude Code plugin

Status: **proposed, not started.** Researched 2026-07-03 against the official Claude Code plugin docs (plugins.md, plugin-marketplaces.md, discover-plugins.md, plugins-reference.md). Not yet implemented — this doc is the plan, the next step is execution.

## Goal

Let a user install both skills with `/plugin marketplace add lucagattoni/TidyClaudeMD` then `/plugin install tidyclaudemd@TidyClaudeMD`, instead of manually cloning and copying `SKILL.md` files into `~/.claude/skills/`.

## Key facts from the plugin spec

- A Claude Code plugin needs a `.claude-plugin/plugin.json` manifest at its root (name, description, version, author, homepage, repository, license). Skills live under `skills/<name>/SKILL.md` inside the plugin — **this repo's existing `skills/` layout already matches that shape**, no directory restructuring needed.
- A **marketplace** is a separate manifest, `.claude-plugin/marketplace.json`, listing catalog entries that each point to a plugin `source` (git repo + path). **A single repo can be both the marketplace and the plugin it lists** — no second repo required.
- Install commands: `/plugin marketplace add owner/repo` (or a local path during dev), then `/plugin install <plugin-name>@<marketplace-name>`.
- **Namespacing**: once installed via plugin, a skill's invocation becomes `/<plugin-name>:<skill-name>` — not the bare `/claude-md-tidy` it is today under a manual copy install. This is an unavoidable, user-visible behavior change for anyone who switches to plugin install.
- **Path portability**: plugins are installed to `~/.claude/plugins/<marketplace>/<plugin>/`, not `~/.claude/skills/`. Any path a skill hardcodes relative to `~/.claude/skills/claude-md-tidy/` will resolve to the wrong location — or not resolve at all — once installed as a plugin.
- Versioning: `plugin.json`'s `version` field is what users see; a marketplace entry's `latest` field points at a git ref (tag or branch). If `plugin.json` has no version, Claude Code treats every commit as a new version, which is worse for anyone who wants deliberate, tagged releases.
- Local testing: `claude --plugin-dir ./` runs a plugin from a local checkout before it's tagged/published anywhere.

## The one real blocker: hardcoded install-location paths

Both `SKILL.md` files currently assume they live at `~/.claude/skills/claude-md-tidy/…` — a manual-copy-install assumption baked into the prose, not just incidental:

- `claude-md-tidy/SKILL.md` Step 7: writes to `~/.claude/skills/claude-md-tidy/RUNS.md` and (per the v0.4.0 archive step) `~/.claude/skills/claude-md-tidy/RUNS-archive/`.
- `claude-md-tidy-reflect/SKILL.md` Step 0: reads `claude-md-tidy/SKILL.md`, `claude-md-tidy/README.md`, `claude-md-tidy/CHANGELOG.md`, `claude-md-tidy/RUNS.md` — all implicitly relative to `~/.claude/skills/`.
- The reflect skill's own frontmatter `description` and Step 5 both reference `~/.claude/skills/claude-md-tidy/SKILL.md` directly.

Under a plugin install, the real path is `~/.claude/plugins/<marketplace>/tidyclaudemd/skills/claude-md-tidy/…`. None of the above resolves correctly there. **This has to be fixed before a plugin release is meaningful** — publishing a plugin whose reflect loop silently can't find its own run records would be worse than not publishing one.

**Proposed fix (do this first, as its own reviewed change — not bundled into the manifest work):** replace every hardcoded `~/.claude/skills/claude-md-tidy/…` reference in both `SKILL.md` files with install-location-agnostic phrasing — e.g. "this skill's own installed directory (`~/.claude/skills/claude-md-tidy/` for a manual copy install, or the plugin's `skills/claude-md-tidy/` subdirectory for a plugin install)" — and verify at implementation time whether Claude Code exposes a runtime variable (something in the spirit of `$PLUGIN_DIR`, unconfirmed by the research pass that produced this plan — check `plugins-reference.md` directly before writing the fix) that skills can reference instead of prose disambiguation. If no such variable exists, the prose fallback above is the correct fix.

## Naming decision

Bundle **both** skills into a single plugin (the plugin spec supports multiple `skills/<name>/` entries per plugin — no need for two separate plugins or two marketplace entries).

Plugin name: **`tidyclaudemd`**, not `claude-md-tidy`. Reasoning: if the plugin were named `claude-md-tidy`, the tidy skill's namespaced invocation would be the redundant `/claude-md-tidy:claude-md-tidy`; naming the plugin after the repo/project instead gives `/tidyclaudemd:claude-md-tidy` and `/tidyclaudemd:claude-md-tidy-reflect` — matches the project's actual public name and reads better.

## Manifest sketch

`.claude-plugin/plugin.json`:
```json
{
  "name": "tidyclaudemd",
  "description": "Self-improving Claude Code skills that keep every repo's CLAUDE.md slim without losing information",
  "version": "0.7.1",
  "author": { "name": "Luca Gattoni" },
  "homepage": "https://github.com/lucagattoni/TidyClaudeMD",
  "repository": "https://github.com/lucagattoni/TidyClaudeMD",
  "license": "UNLICENSED"
}
```
`license` is a placeholder — no `LICENSE` file exists in the repo yet. Pick one (or explicitly stay unlicensed/all-rights-reserved) before publishing; this plan doesn't decide that.

`.claude-plugin/marketplace.json`:
```json
{
  "version": "1",
  "plugins": [
    {
      "name": "tidyclaudemd",
      "description": "Self-improving Claude Code skills that keep every repo's CLAUDE.md slim without losing information",
      "author": { "name": "Luca Gattoni" },
      "homepage": "https://github.com/lucagattoni/TidyClaudeMD",
      "source": { "type": "git", "url": "lucagattoni/TidyClaudeMD", "path": "." },
      "latest": "v0.7.1"
    }
  ]
}
```
The `latest` field needs to track the newest release tag — see the versioning-integration item below.

## Versioning integration

`claude-md-tidy-reflect`'s Step 5 (Apply) already bumps the suite version in both `SKILL.md` frontmatters and the README header on every self-improvement pass. Once the plugin manifests exist, that same step needs two more sub-steps:
1. Update `.claude-plugin/plugin.json`'s `version` field to match.
2. Update `.claude-plugin/marketplace.json`'s `latest` field to the new release tag, once the tier/release-tagging convention from this session (`v<version>` git tags) continues.

This is a real edit to `claude-md-tidy-reflect/SKILL.md` Step 5 — not done in this plan doc, listed here so it isn't forgotten when the manifests land.

## Phased action items (execution order)

0. **Write `install.sh`** (above). No path-portability risk, no dependency on the plugin work — can ship independently and immediately, and gives manual-install users a real one-liner in the meantime.
1. **Fix path portability** in both `SKILL.md` files (the blocker above). Review like any other tier change before committing.
2. **Add `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`** per the sketches above.
3. **Extend `claude-md-tidy-reflect` Step 5** with the two versioning-integration sub-steps above.
4. **Local test**: `claude --plugin-dir ./` against a scratch repo, confirm both skills trigger under their namespaced names and that Step 7/archive writes land in the right place.
5. **Tag and verify the marketplace flow end-to-end**: from a *different* machine/checkout, run `/plugin marketplace add lucagattoni/TidyClaudeMD` then `/plugin install tidyclaudemd@TidyClaudeMD`, confirm both skills are usable.
6. **Update README's Install section** (added separately, see below) to document all three paths — `install.sh`, manual copy, and plugin — with the namespacing difference (`/tidyclaudemd:claude-md-tidy` vs. bare `/claude-md-tidy`) flagged explicitly so no one is surprised by it.

## Manual-install script (for users who don't want the plugin)

The plugin path isn't the only distribution option worth having — some users won't want marketplace auto-updates, won't want the longer `/tidyclaudemd:claude-md-tidy` namespaced form, or just want a one-liner instead of `git clone` + manual copy. Keep both paths, not plugin-only.

**Proposed: `install.sh` at the repo root.** A small, dependency-free shell script:
1. Resolves `~/.claude/skills/` (create it if missing).
2. Copies `skills/claude-md-tidy/` and `skills/claude-md-tidy-reflect/` into it wholesale (so `RUNS.md`/`RUNS-archive/` that already exist there from prior use are left alone — copy the `SKILL.md` files, don't wipe the directory).
3. Detects an existing install and reports old → new version (read the `version:` frontmatter before overwriting) rather than silently clobbering.
4. Prints a short confirmation: which skills were installed/updated, and the invocation form (`/claude-md-tidy`, `/claude-md-tidy-reflect` — no namespace, since this is a manual-copy install, not a plugin install).

Usable both as a one-time run (`curl`'d or cloned-then-run) and as the same mechanism the README's manual-install instructions point to — replacing today's "copy it back to the installed location by hand" step with one command. This is independent of the plugin work above and can ship first, since it has none of the path-portability risk (it *is* the manual-copy layout the skills already assume).

## What this plan does not decide

- Whether to keep the manual-copy method documented indefinitely after the plugin ships, or eventually deprecate it. Manual-copy gives the short `/claude-md-tidy` invocation; plugin install gives auto-update-via-marketplace at the cost of the longer namespaced form. Worth an explicit user decision once the plugin actually works, not guessed here.
- Licensing (no `LICENSE` file exists yet; `plugin.json`'s `license` field needs a real answer before a public release, separate from any repo-visibility question).
