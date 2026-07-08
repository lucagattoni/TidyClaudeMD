# nick2781/claudoctor

**URL:** https://github.com/nick2781/claudoctor
**Stars (2026-07-09):** 0 (created 2026-05-22, still active — pushed 2026-05-25, CalVer-versioned, roadmap items landing)
**Category:** Direct competitor — spans `--skills` and CLAUDE.md classes; also the only tool found that scans *outside* Claude Code entirely
**License:** MIT
**Distribution:** `npx claudoctor skills` (zero-install) or `npm install -g claudoctor`; a pure Node CLI, no Claude Code skill/plugin wrapper — never runs inside a Claude Code session unless you count its optional Anthropic-API cross-check

## What it is

"The lint tool for agent skills and CLAUDE.md." Despite zero stars, this is the most structurally sophisticated single competitor found in this search — and the only one that is explicitly **cross-agent**: it scans skill directories for Claude Code, Codex, Hermes, and Cursor in one pass, not just Claude's.

## What it does

### `claudoctor skills` — cross-agent skill audit

Scans every known skill location on disk at once:

| Agent | Path(s) scanned |
|---|---|
| claude | `~/.claude/skills/`, `~/.claude/plugins/cache/`, `~/.claude/plugins/marketplaces/` |
| codex | `~/.codex/skills/` |
| hermes | `~/.hermes/skills/` |
| cursor | `~/.cursor/rules/`, `$PWD/.cursor/rules/` |
| project | `$PWD/.claude/skills/` |

For every skill found, it reports:
- **Token rank** — exact token cost per skill via `@anthropic-ai/tokenizer` (the real tokenizer, not an estimate).
- **Duplicates** — byte-identical `SKILL.md` content, even across different agents' directories.
- **Near-duplicates** — identical body, different frontmatter ("frontmatter drift" — the same procedure copy-pasted with a renamed `name:`).
- **Conflicts** — same skill `name`, different content (a genuine collision, not just wasted tokens).
- **Overlap** — semantically similar `name` + `description` via Jaccard similarity on token sets; `--deep` also compares full body tokens (O(N²), opt-in for the slower pass).
- **Savings estimate** — the tokens reclaimable by removing duplicates, near-duplicates, and the smaller half of every overlap pair, with double-counting explicitly avoided.

A real run on a heavily-used machine: *"Scanned 393 skills, 2462.2k tokens total... Duplicates (55), Near-duplicates (5), Conflicts (57)... Estimated savings: ~499.0k tokens."* This is the scale problem TidyClaudeMD's `--skills` class doesn't yet address — it interrogates one repo's skills, not a whole machine's accumulated skill sprawl across every tool and plugin marketplace cache.

### `claudoctor claudemd` — static + optional LLM diagnosis

A rule engine (declarative rules in `rules.data.ts`) checks for:
- Token bloat (warn at 5,000 tokens, error at 15,000).
- Rule overload — too many imperative `MUST`/`NEVER` directives in one file.
- Verbose/vague language — weasel words like "appropriate," "where suitable."
- Counterproductive patterns — phrasing known to hurt agent behavior.
- Conflicts — contradictory rules within the same section.
- **Missing best-practice sections** — Tone, Tools, Workflow, etc. A *completeness* check, not just a slimming one; nothing in TidyClaudeMD asks "does this file have the sections a good CLAUDE.md should have?"
- Structural issues — missing frontmatter, broken headings, emphasis spam.

The LLM cross-check is opt-in and *outside* any Claude Code session: if `ANTHROPIC_API_KEY` is set, claudoctor calls the Anthropic API directly (defaulting to a specified model, e.g. `claude-haiku-4-5-20251001`) to catch what the static rules miss, then falls back to rules-only with a stderr warning if no key is present. This is a genuinely different distribution model — a CI job or pre-commit hook can run the full LLM-assisted diagnosis with zero Claude Code installation, paying only for the API call.

### `claudoctor report` — shareable team artifact

Combines both analyses into one report, in Markdown, JSON, or a **single offline HTML file** (inline CSS/JS, dark-mode via `prefers-color-scheme`, severity counts, three review columns for CLAUDE.md findings / skills findings / duplicates, with `file://` links back to source). Built explicitly for team review, not just an individual's terminal — a distribution shape TidyClaudeMD has no equivalent of (its output is conversational, per-session, and only persisted as the internal run log).

### `claudoctor skill add/list/remove` — a skill-pack package manager

A separate, explicitly-networked feature: installs community skill packs from a git URL, GitHub shorthand, or a named registry entry into `~/.claudoctor/skills/<pack-name>/`, with confirmation prompts (or `--yes` for CI) and a pluggable registry URL. Not a hygiene feature — closer to a plugin manager — but notable as a second axis ("install more skills," not "tidy the skills you have") that this niche's tools are starting to combine into one CLI.

## Gaps relative to TidyClaudeMD

- No repo-comprehension gate on the `claudemd` side — the static rules never verify a claim against the actual repo (a referenced command, a stale file path); only the *optional*, unverified LLM pass could catch that, and even then it's a single opaque cross-check, not a structured per-line test.
- No CHALLENGE-equivalent — findings are severity-tagged (warn/error) and reported; there's no per-item, evidence-cited question posed back to the user with resolution options.
- No self-improvement loop — the rule engine is static and versioned by hand (CalVer releases), not evidence-driven from real scan outcomes.
- No auto memory coverage at all.
- Auto-fix/auto-merge for duplicates is explicitly marked "Next CalVer release" in its own roadmap — not shipped yet as of this check, so its reversibility story for *applied* changes is currently thinner than the diagnosis story.
