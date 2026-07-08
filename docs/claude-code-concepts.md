# Claude Code file types — a primer

New to Claude Code? This page explains the four kinds of files TidyClaudeMD maintains, in plain terms, before you read anything else in this repo. For the authoritative detail on any of them, follow the official docs links below — this page stays intentionally short.

## `CLAUDE.md` — instructions you write

A markdown file with instructions for Claude: build commands, conventions, "always do X" rules. Claude reads it automatically at the start of every session — you never have to remind it. It can live in several places at once (all get combined):

- `./CLAUDE.md` or `./.claude/CLAUDE.md` — project-level, shared with your team via git.
- `~/.claude/CLAUDE.md` — personal, applies to every project on your machine.
- `./CLAUDE.local.md` — personal preferences for one specific project, gitignored.

Official docs: [code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory)

## `.claude/rules/*.md` — CLAUDE.md, split into topic files

For a project whose CLAUDE.md is getting long, you can split it into separate files under `.claude/rules/` (e.g. `testing.md`, `security.md`). A rule file can optionally declare `paths:` frontmatter so it only loads into context when Claude is working with matching files — a way to keep instructions available without paying their token cost every session.

Official docs: [code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory) (see "Organize rules with `.claude/rules/`")

## `SKILL.md` — a packaged procedure

A skill bundles a reusable, multi-step workflow into its own file (`.claude/skills/<name>/SKILL.md`). Unlike CLAUDE.md, a skill's full instructions aren't loaded every session — only its short description is, and Claude reads the rest only when the task actually calls for it (a pattern called "progressive disclosure"). This is why procedures belong in a skill, not CLAUDE.md: they're cheaper to keep around.

Official docs: [code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills)

## `MEMORY.md` — notes Claude writes about itself

Auto memory is the one file type here Claude writes, not you. As you correct it or it discovers something worth remembering (a build quirk, a debugging insight), it saves a note to `~/.claude/projects/<project>/memory/`. A compact index (`MEMORY.md`) loads every session; detailed topic files load only when needed. It's the "learned, ad hoc" counterpart to CLAUDE.md's "written, deliberate" instructions.

Official docs: [code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory) (see "Auto memory")

## Where TidyClaudeMD fits

All four file types above are context Claude loads every session, or nearly every session — so they all cost tokens and get harder to maintain as they grow. TidyClaudeMD is a Claude Code plugin (two skills, described in the [README](../README.md)) that audits and slims all four, relocating content to its right home instead of just deleting it. Full mechanics: [reference.md](reference.md).
