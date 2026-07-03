# Plugin packaging plan — installable Claude Code plugin

Status: **proposed, not started.** Researched 2026-07-03 against the official Claude Code plugin docs (plugins.md, plugin-marketplaces.md, discover-plugins.md, plugins-reference.md). Not yet implemented — this doc is the plan, the next step is execution.

_Updated 2026-07-03: incorporated three user decisions — skills are renamed `claudemd-tidy` / `claudemd-tidy-reflect` as part of this work (not just the plugin wrapper); distribution is **plugin-only**, no manual-install fallback and no `install.sh`; license is **MIT** (see `LICENSE` at repo root, added this pass). Also confirmed, via a targeted doc check, the exact runtime path-resolution mechanism the earlier draft had left as an open question: `${CLAUDE_PLUGIN_ROOT}` and `${CLAUDE_PLUGIN_DATA}`, both substituted inline in skill content — see the blocker section below for which one applies where._

## Goal

Let a user install both skills with `/plugin marketplace add lucagattoni/TidyClaudeMD` then `/plugin install tidyclaudemd@TidyClaudeMD`. This will be the **only** supported install path once it ships — no manual copy, no standalone script.

## Key facts from the plugin spec

- A Claude Code plugin needs a `.claude-plugin/plugin.json` manifest at its root (name, description, version, author, homepage, repository, license). Skills live under `skills/<name>/SKILL.md` inside the plugin — this repo's existing `skills/` layout already matches that shape, no directory restructuring needed beyond the rename below.
- A **marketplace** is a separate manifest, `.claude-plugin/marketplace.json`, listing catalog entries that each point to a plugin `source` (git repo + path). **A single repo can be both the marketplace and the plugin it lists** — no second repo required.
- Install commands: `/plugin marketplace add owner/repo` (or a local path during dev), then `/plugin install <plugin-name>@<marketplace-name>`.
- **Namespacing**: once installed via plugin, a skill's invocation becomes `/<plugin-name>:<skill-name>` — e.g. `/tidyclaudemd:claudemd-tidy`, not a bare `/claudemd-tidy`. Since there is no manual-install fallback, this namespaced form is simply *the* invocation — not a tradeoff against a shorter alternative.
- **Path portability**: plugins are installed to `~/.claude/plugins/<marketplace>/<plugin>/`. Any path a skill hardcodes relative to `~/.claude/skills/…` (today's manual-install assumption) is wrong under a plugin install. See the blocker section below.
- Versioning: `plugin.json`'s `version` field is what users see; a marketplace entry's `latest` field points at a git ref (tag or branch). If `plugin.json` has no version, Claude Code treats every commit as a new version, which is worse for anyone who wants deliberate, tagged releases.
- Local testing: `claude --plugin-dir ./` runs a plugin from a local checkout before it's tagged/published anywhere — this is the development-time equivalent of "installing" while iterating, and is why dropping manual install doesn't strand development: local testing doesn't need it.

## Skill rename (part of this work, not yet executed)

Both skills are renamed as part of shipping the plugin — this is a real rename of the live, already-released skills, not just new names for a wrapper:

| Current | New |
|---|---|
| `claude-md-tidy` (dir `skills/claude-md-tidy/`) | `claudemd-tidy` (dir `skills/claudemd-tidy/`) |
| `claude-md-tidy-reflect` (dir `skills/claude-md-tidy-reflect/`) | `claudemd-tidy-reflect` (dir `skills/claudemd-tidy-reflect/`) |

This touches: both directory names, both `SKILL.md` frontmatter `name:` fields, every internal cross-reference between the two skills (the reflect skill's Step 0/1/3 all name the tidy skill's files by path), every README section header/path/table row, the CHANGELOG's suite-description line, and the manifest sketches below. **Execute this as its own reviewed, version-bumped change** (same discipline as the four backlog tiers earlier) — it is a rename with real blast radius (anyone who manually copied the old names has stale directories), not a cosmetic edit.

## The one real blocker: hardcoded install-location paths

Both `SKILL.md` files currently assume they live at `~/.claude/skills/claude-md-tidy/…`:

- `claude-md-tidy/SKILL.md` Step 7: writes to `~/.claude/skills/claude-md-tidy/RUNS.md` and (per the v0.4.0 archive step) `~/.claude/skills/claude-md-tidy/RUNS-archive/`.
- `claude-md-tidy-reflect/SKILL.md` Step 0: reads `claude-md-tidy/SKILL.md`, `claude-md-tidy/README.md`, `claude-md-tidy/CHANGELOG.md`, `claude-md-tidy/RUNS.md`.
- The reflect skill's own frontmatter `description` and Step 5 reference `~/.claude/skills/claude-md-tidy/SKILL.md` directly.

Since manual install is being dropped entirely, these only ever need to resolve **one** way going forward: the plugin's actual installed path. **Confirmed** (Claude Code docs, `plugins-reference.md`, "Environment variables"): Claude Code substitutes two variables inline in skill content itself, before the model sees the text — no shell/hook required:

- **`${CLAUDE_PLUGIN_ROOT}`** — absolute path to the plugin's installation directory (the current version's bundled files).
- **`${CLAUDE_PLUGIN_DATA}`** — resolves to `~/.claude/plugins/data/{id}/`, and **persists across plugin version updates**.

These are not interchangeable for this fix. `RUNS.md`/`RUNS-archive/` are exactly the kind of state that must survive a plugin update (a user updating `tidyclaudemd` to a new version must not lose their run-record history) — those belong under `${CLAUDE_PLUGIN_DATA}`, not `${CLAUDE_PLUGIN_ROOT}`, which likely gets replaced wholesale on update along with the rest of the bundled skill files. The skill's own reference docs (`SKILL.md`, `README.md`, `CHANGELOG.md` — read-only, meant to reflect whatever version is currently installed) belong under `${CLAUDE_PLUGIN_ROOT}`.

**Proposed fix:** replace every hardcoded `~/.claude/skills/claude-md-tidy/…` path in both `SKILL.md` files:
- `${CLAUDE_PLUGIN_DATA}/RUNS.md` and `${CLAUDE_PLUGIN_DATA}/RUNS-archive/` (was `~/.claude/skills/claude-md-tidy/RUNS.md` / `RUNS-archive/`).
- `${CLAUDE_PLUGIN_ROOT}/skills/claudemd-tidy/SKILL.md` for the reflect skill's cross-references to the tidy skill's `SKILL.md`/`README.md`/`CHANGELOG.md` — `${CLAUDE_PLUGIN_ROOT}` is the whole **plugin's** root per the docs ("your plugin's installation directory," not a per-skill directory), and since both skills are bundled in the same `tidyclaudemd` plugin, it's shared context either skill can reference the other from; no `../` needed. Confirm this against the actual installed layout at implementation time — it's a reasonable reading of the doc language, not independently verified by running a real install.

## Manifest sketch

`.claude-plugin/plugin.json`:
```json
{
  "name": "tidyclaudemd",
  "description": "Self-improving Claude Code skills that keep every repo's CLAUDE.md slim without losing information",
  "version": "0.7.3",
  "author": { "name": "Luca Gattoni" },
  "homepage": "https://github.com/lucagattoni/TidyClaudeMD",
  "repository": "https://github.com/lucagattoni/TidyClaudeMD",
  "license": "MIT"
}
```

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
      "latest": "v0.7.3"
    }
  ]
}
```
The `latest` field needs to track the newest release tag — see the versioning-integration item below.

## Versioning integration

`claudemd-tidy-reflect`'s Step 5 (Apply) already bumps the suite version in both `SKILL.md` frontmatters and the README header on every self-improvement pass. Once the plugin manifests exist, that same step needs two more sub-steps:
1. Update `.claude-plugin/plugin.json`'s `version` field to match.
2. Update `.claude-plugin/marketplace.json`'s `latest` field to the new release tag, continuing the `v<version>` git-tag convention already in use.

This is a real edit to `claudemd-tidy-reflect/SKILL.md` Step 5 — not done in this plan doc, listed here so it isn't forgotten when the manifests land.

## Phased action items (execution order)

1. **Rename both skills** (table above) — directories, frontmatter, every cross-reference in both `SKILL.md` files, README, and CHANGELOG's suite-description line. Reviewed and version-bumped on its own, before anything else below.
2. **Fix path portability** in both `SKILL.md` files against the renamed directories, using `${CLAUDE_PLUGIN_DATA}` for `RUNS.md`/`RUNS-archive/` and `${CLAUDE_PLUGIN_ROOT}` for the read-only cross-references, per the confirmed mechanism above. Reviewed like any other tier change before committing.
3. **Add `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`** per the sketches above.
4. **Extend `claudemd-tidy-reflect` Step 5** with the two versioning-integration sub-steps above.
5. **Local test**: `claude --plugin-dir ./` against a scratch repo, confirm both skills trigger under their namespaced names and that Step 7/archive writes land in the right place.
6. **Tag and verify the marketplace flow end-to-end**: from a *different* machine/checkout, run `/plugin marketplace add lucagattoni/TidyClaudeMD` then `/plugin install tidyclaudemd@TidyClaudeMD`, confirm both skills are usable.
7. **Update README's Install section** to document the plugin command as the sole install path, with the namespaced invocation (`/tidyclaudemd:claudemd-tidy`) shown as *the* form, not an alternative to something shorter.

## What this plan does not decide

- Whether `${CLAUDE_PLUGIN_ROOT}/skills/claudemd-tidy/…` is exactly right for the reflect skill's cross-reference to its sibling skill — the reasoning holds (see above) but hasn't been verified against a real install; confirm during step 2, don't take it on faith.
- Nothing else is open: distribution is plugin-only (no manual-copy fallback, no `install.sh`), licensing is MIT (`LICENSE` at repo root), and the `${CLAUDE_PLUGIN_ROOT}` / `${CLAUDE_PLUGIN_DATA}` split for path portability is confirmed, not speculative.
