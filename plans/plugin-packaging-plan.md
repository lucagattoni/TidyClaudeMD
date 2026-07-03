# Plugin packaging plan — installable Claude Code plugin

Status: **phases 1-4 executed, in v0.8.0.** Researched 2026-07-03 against the official Claude Code plugin docs (plugins.md, plugin-marketplaces.md, discover-plugins.md, plugins-reference.md). Phases 5-7 (local test, external marketplace-flow verification, this doc's own cleanup) are what remain — see the phased list below for current status per item.

_Updated 2026-07-03 (second pass): executed phases 1-4 in the same release, per explicit user request — rename, path-portability fix, manifest files, and the reflect skill's versioning-integration sub-step are all done and committed. The "skill rename" and "one real blocker" sections below are now historical design rationale for a change already made, not a to-do — kept for reference since the reasoning (why `${CLAUDE_PLUGIN_DATA}` vs. `${CLAUDE_PLUGIN_ROOT}`, why rename at all) is still worth having on hand._

_Updated 2026-07-03 (first pass): incorporated three user decisions — skills are renamed `claudemd-tidy` / `claudemd-tidy-reflect` as part of this work (not just the plugin wrapper); distribution is **plugin-only**, no manual-install fallback and no `install.sh`; license is **MIT** (see `LICENSE` at repo root, added this pass). Also confirmed, via a targeted doc check, the exact runtime path-resolution mechanism the earlier draft had left as an open question: `${CLAUDE_PLUGIN_ROOT}` and `${CLAUDE_PLUGIN_DATA}`, both substituted inline in skill content — see the blocker section below for which one applies where._

## Goal

Let a user install both skills with `/plugin marketplace add lucagattoni/TidyClaudeMD` then `/plugin install tidyclaudemd@TidyClaudeMD`. This is the **only** supported install path — no manual copy, no standalone script. The manifests exist as of v0.8.0; what's not yet done is independent, external verification that the flow actually works end to end (phase 6 below).

## Key facts from the plugin spec

- A Claude Code plugin needs a `.claude-plugin/plugin.json` manifest at its root (name, description, version, author, homepage, repository, license). Skills live under `skills/<name>/SKILL.md` inside the plugin — this repo's existing `skills/` layout already matches that shape, no directory restructuring needed beyond the rename below.
- A **marketplace** is a separate manifest, `.claude-plugin/marketplace.json`, listing catalog entries that each point to a plugin `source` (git repo + path). **A single repo can be both the marketplace and the plugin it lists** — no second repo required.
- Install commands: `/plugin marketplace add owner/repo` (or a local path during dev), then `/plugin install <plugin-name>@<marketplace-name>`.
- **Namespacing**: once installed via plugin, a skill's invocation becomes `/<plugin-name>:<skill-name>` — e.g. `/tidyclaudemd:claudemd-tidy`, not a bare `/claudemd-tidy`. Since there is no manual-install fallback, this namespaced form is simply *the* invocation — not a tradeoff against a shorter alternative.
- **Path portability**: plugins are installed to `~/.claude/plugins/<marketplace>/<plugin>/`. Any path a skill hardcodes relative to `~/.claude/skills/…` (today's manual-install assumption) is wrong under a plugin install. See the blocker section below.
- Versioning: `plugin.json`'s `version` field is what users see; a marketplace entry's `latest` field points at a git ref (tag or branch). If `plugin.json` has no version, Claude Code treats every commit as a new version, which is worse for anyone who wants deliberate, tagged releases.
- Local testing: `claude --plugin-dir ./` runs a plugin from a local checkout before it's tagged/published anywhere — this is the development-time equivalent of "installing" while iterating, and is why dropping manual install doesn't strand development: local testing doesn't need it.

## Skill rename (done, v0.8.0)

Both skills are renamed as part of shipping the plugin — this is a real rename of the live, already-released skills, not just new names for a wrapper:

| Current | New |
|---|---|
| `claude-md-tidy` (dir `skills/claude-md-tidy/`) | `claudemd-tidy` (dir `skills/claudemd-tidy/`) |
| `claude-md-tidy-reflect` (dir `skills/claude-md-tidy-reflect/`) | `claudemd-tidy-reflect` (dir `skills/claudemd-tidy-reflect/`) |

This touches: both directory names, both `SKILL.md` frontmatter `name:` fields, every internal cross-reference between the two skills (the reflect skill's Step 0/1/3 all name the tidy skill's files by path), every README section header/path/table row, the CHANGELOG's suite-description line, and the manifest sketches below. **Execute this as its own reviewed, version-bumped change** (same discipline as the four backlog tiers earlier) — it is a rename with real blast radius (anyone who manually copied the old names has stale directories), not a cosmetic edit.

## The one real blocker: hardcoded install-location paths (fixed, v0.8.0)

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

## Manifests (real files now, v0.8.0)

No longer a sketch — `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` exist at repo root with this shape (version numbers tracked live, not reproduced here to avoid a second place to keep in sync — see the actual files).

The `marketplace.json` `latest` field needs to track the newest release tag — see the versioning-integration item below, now implemented as `claudemd-tidy-reflect` Step 5 sub-step 7.

## Versioning integration (done, v0.8.0)

`claudemd-tidy-reflect` Step 5 sub-step 7 now keeps `.claude-plugin/plugin.json`'s `version` field and `.claude-plugin/marketplace.json`'s `latest` field in sync with every future version bump, continuing the `v<version>` git-tag convention already in use.

## Phased action items (execution order)

1. ~~**Rename both skills**~~ **Done, v0.8.0.**
2. ~~**Fix path portability**~~ **Done, v0.8.0.**
3. ~~**Add `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`**~~ **Done, v0.8.0.**
4. ~~**Extend `claudemd-tidy-reflect` Step 5**~~ **Done, v0.8.0.**
5. **Local test**: `claude --plugin-dir ./` against a scratch repo, confirm both skills trigger under their namespaced names and that Step 7/archive writes land in the right place. **Done, v0.8.0** — `/tidyclaudemd:claudemd-tidy` loaded and triggered correctly via `--plugin-dir`; `--report` mode ran preflight (encryption/CI/visibility checks) and produced the correct mechanical-checks + labeled-unverified-impression output; `${CLAUDE_PLUGIN_DATA}` resolved to a real path (`~/.claude/plugins/data/tidyclaudemd-inline/RUNS.md` — the `-inline` suffix is Claude Code's own naming for a locally-loaded `--plugin-dir` plugin, not a marketplace-installed one) outside the target project, confirming the path-portability fix works as designed. The actual `RUNS.md` write was blocked by this test session's own sandbox restrictions on cross-directory writes in non-interactive mode — a property of the test harness, not the plugin — so the write itself is unverified; the *resolved path* is confirmed correct. The reflect skill's sibling-skill cross-reference (`${CLAUDE_PLUGIN_ROOT}/skills/claudemd-tidy/…`) was not separately exercised in this pass.
6. **Tag and verify the marketplace flow end-to-end**: from a *different* machine/checkout, run `/plugin marketplace add lucagattoni/TidyClaudeMD` then `/plugin install tidyclaudemd@TidyClaudeMD`, confirm both skills are usable. **Not done** — needs a genuinely separate environment, not something verifiable from within the same session/machine that authored the plugin.
7. ~~**Update README's Install section**~~ **Done, v0.8.0.**

## What this plan does not decide

- Whether `${CLAUDE_PLUGIN_ROOT}/skills/claudemd-tidy/…` is exactly right for the reflect skill's cross-reference to its sibling skill — the reasoning holds (see above) but hasn't been verified against a real install; confirm during step 5's local test, don't take it on faith.
- Nothing else is open: distribution is plugin-only (no manual-copy fallback, no `install.sh`), licensing is MIT (`LICENSE` at repo root), and the `${CLAUDE_PLUGIN_ROOT}` / `${CLAUDE_PLUGIN_DATA}` split for path portability is confirmed, not speculative.
