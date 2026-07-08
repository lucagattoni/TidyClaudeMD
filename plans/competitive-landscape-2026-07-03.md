# Competitive landscape — CLAUDE.md maintenance tools

Source: GitHub search (`gh search repos`) across `CLAUDE.md optimize/tidy/slim/cleanup`, `AGENTS.md maintain`, `claude memory hygiene`, plus direct repo inspection (`gh repo view`) of every plausible match. Triggered by the repo rename to `TidyClaudeMD`, to sanity-check whether the name and positioning collide with an existing tool. Confirmed: `lucagattoni/TidyClaudeMD` is the current public name, visibility PUBLIC.

**Bottom line (2026-07-03): no name collision, but a crowded and well-developed niche.** At least four repos solve the identical problem ("CLAUDE.md is bloated, shrink it losslessly").

**Purpose of this document.** For each project worth documenting — selected because it's *relevant to improving TidyClaudeMD*, not merely because it exists — state plainly what TidyClaudeMD lacks relative to it. Every gap below is a real, shipped capability in another tool that TidyClaudeMD does not currently have.

---

_**Updated 2026-07-09** against TidyClaudeMD v0.20.2 (up from v0.9.1 at the original pass — scope expanded from CLAUDE.md-only to four target classes: project/user CLAUDE.md, `.claude/rules/`, `SKILL.md` skills, and auto memory, plus a self-improving reflect loop and a git-commit-per-iteration reversibility gate). Re-checked all 9 previously catalogued repos (all still active, none archived) and closed out two gaps the original pass had flagged (encryption-awareness, CI-dependency scanning — both shipped in v0.6.0). Searched fresh angles matching the expanded scope and found **two substantial new competitors**, both scoped to the `--skills` class: `TheStack-ai/pulser` and `nick2781/claudoctor`. Three more repos surfaced with near-zero signal and no gap worth recording. Ranked gap list: [Gaps to close, ranked by evidence](#gaps-to-close-ranked-by-evidence-updated-2026-07-09)._

---

## Direct competitors (same problem: shrink a bloated CLAUDE.md)

### `wrsmith108/claude-md-optimizer` (19 stars, active — pushed 2026-06-25)
Progressive-disclosure optimizer for CLAUDE.md, AGENTS.md, and copilot-instructions.md in one tool — a 6-phase workflow (detect → constraints → plan → review → execute → validate), using each format's own native sub-doc mechanism rather than generic markdown links.

**Gap:** TidyClaudeMD only handles Claude Code's own CLAUDE.md — it detects whether AGENTS.md is *imported* but never audits or slims AGENTS.md's own content, and has no coverage of copilot-instructions.md at all. wrsmith108 covers all three in one pass.

**Gap:** wrsmith108 published a measured eval of its rich-abstract-pointer pattern (thin link → agent re-opened the sub-doc 5/5 times on test questions; rich abstract → 1-2/5). TidyClaudeMD mandates the identical pattern by rule (RELOCATE's rich-abstract pointer) but has never measured whether its own output achieves the same reduction — the design assumption is unverified.

### `geuneda/claude-md-optimizer` (2 stars, dormant since 2026-03-18)
Scoring-and-linting-first: a 0-100 automated score, regex anti-pattern detection, non-English token-overhead detection, session-cost estimation, and a standalone Python script that runs the whole analysis outside any agent session.

**Gap:** geuneda's analysis script needs no LLM session at all — it's a free, instant, CI-runnable check. TidyClaudeMD's closest equivalent, `--report` mode, still only ever runs inside a live Claude Code session; there is no standalone entry point.

**Gap:** geuneda's token-overhead figures are still heuristic like TidyClaudeMD's own, but its non-English detection and per-file scoring are packaged as a `--json` machine-readable output for automation — TidyClaudeMD's report mode has no machine-readable output mode.

### `NicoAcosta/claude-md-optimizer` (0 stars, dormant since 2026-04-05)
Simple 4-phase workflow, hard target <50 lines. Distributed via skills.sh, a plugin marketplace, and support for Cursor, Codex/OpenCode, and Gemini CLI in the same package.

**Gap:** NicoAcosta installs into four different agent ecosystems from one package. TidyClaudeMD is a Claude-Code-only plugin, with no distribution path to Cursor, Codex, OpenCode, or Gemini CLI users at all.

### `Pinkers01/claude-md-optimizer` (0 stars, dormant since 2026-05-06)
A standalone GUI app (macOS/.app, Windows/.exe, or a single HTML file), not a Claude Code skill — 100% client-side, drag-and-drop keep/move/delete, duplicate + contradiction detection against a small hardcoded conflict-pair library (e.g. "Stripe vs Mollie").

**Gap:** Pinkers01 works over *any* pasted LLM context file with zero install and no Claude Code dependency — a user who doesn't use Claude Code at all, or who just wants a one-off check, has no equivalent path into TidyClaudeMD, which requires a full plugin install and a live session.

**Gap:** Pinkers01 ships an explicit library of known contradiction patterns (specific named conflicts, not just shapes). TidyClaudeMD's Consistent? test probes contradiction *shapes* (autonomy vs. ask-first, etc.) but has no equivalent of a maintained, explicit pattern library for domain-specific recurring conflicts.

### `tsalkin/claude-memory-hygiene` (1 star, active — pushed 2026-06-19)
Targets `MEMORY.md` specifically — the same artifact TidyClaudeMD's `--memory` class now covers. Zero-dependency Python CLI, dry-run by default, archives (never deletes) stale files using a pin/volatile regex split, no LLM judgment involved.

**Gap:** tsalkin runs with zero session cost — a cron job or pre-commit hook, no LLM call at all. TidyClaudeMD's `--memory` class always requires a live Claude Code session to run any check, cheap or not.

### `TheStack-ai/pulser` (18 stars, active — pushed 2026-06-30) — **new, 2026-07-09**
Full doc: [`competitors/thestack-ai-pulser.md`](competitors/thestack-ai-pulser.md). A dedicated `SKILL.md` linter with a standout feature: **`pulser eval`** actually *runs* a skill against YAML-defined test cases via `claude -p` and asserts on the output, tracking regressions across runs with a distinct exit code from a fresh failure.

**Gap:** TidyClaudeMD's `--skills` class only checks structural hygiene (description quality, progressive disclosure, frontmatter) — it has no way to verify a skill actually *behaves* correctly, and no way to detect that a previously-working skill has silently regressed. pulser does both.

**Gap:** pulser's reversibility (single-level backup + `undo`) works with zero git dependency. TidyClaudeMD's reversibility gate requires the target to be git-tracked to apply autonomously at all — an ungated, non-git target always falls back to confirm-first, with no lighter-weight safety net in between.

**Gap:** pulser ships as a real GitHub Action on the Marketplace, with documented exit codes (`0`/`1`/`2`, `--strict` promotes warnings to errors) built to gate a pull request. TidyClaudeMD has no CI-native distribution at all.

### `nick2781/claudoctor` (0 stars, active — created 2026-05-22, roadmap items still landing) — **new, 2026-07-09**
Full doc: [`competitors/nick2781-claudoctor.md`](competitors/nick2781-claudoctor.md). `claudoctor skills` scans **cross-agent** (Claude, Codex, Hermes, Cursor) skill directories — including plugin-marketplace caches — in one pass: exact-tokenizer token ranking, byte-identical duplicates, near-duplicates, name conflicts, and similarity-based overlap, with a savings estimate. `claudoctor report --format html` renders a single offline, shareable file.

**Gap:** claudoctor scans four agent ecosystems in one pass; TidyClaudeMD only ever looks at Claude Code's own directories.

**Gap:** claudoctor sweeps `~/.claude/plugins/cache/` and `~/.claude/plugins/marketplaces/`, not just live skill directories — and its own example run found real scale there (55 duplicates, 57 conflicts across 393 skills). TidyClaudeMD's `--skills --user` never looks at plugin cache/marketplace directories at all. Not hypothetical for this repo: mid-session on 2026-07-08, this exact machine's `~/.claude/plugins/cache/tidyclaudemd/tidyclaudemd/` was found holding two full stale copies of TidyClaudeMD itself (`0.9.1/` and `0.18.0/`) — a gap a TidyClaudeMD run would never have surfaced on its own.

**Gap:** claudoctor computes exact per-file token counts via `@anthropic-ai/tokenizer`. TidyClaudeMD's size/cost figures (session-cost estimate, non-English overhead) are all heuristic estimates, never a real tokenizer call.

**Gap:** claudoctor detects cross-skill duplication (byte-identical, near-duplicate via frontmatter drift, name conflicts, semantic overlap via Jaccard similarity). TidyClaudeMD's `--skills` class interrogates each `SKILL.md` file in isolation — there is no cross-file test at all.

**Gap:** claudoctor's `claudemd` command flags missing canonical sections (Tone, Tools, Workflow) — a completeness check. TidyClaudeMD only ever asks "should this content stay, move, or go" — it has no check for content that should exist but doesn't.

**Gap:** claudoctor's `report --format html` produces a shareable, offline document for team review. TidyClaudeMD's only durable output is the machine-oriented run log — nothing renders a human-shareable report.

---

## Minor / low-signal mentions (2026-07-09, no dedicated docs)

Found via the same search sweep; CLAUDE.md optimization is a small feature within a broader tool for two of these, all at 0-1 stars with no recent momentum, and none exposed a gap worth recording:

| Repo | Stars | What it actually does |
|---|---|---|
| `sgamma/skills` | 0 | Personal skill collection; one of the skills is a progressive-disclosure CLAUDE.md optimizer |
| `BENZEMA216/self-purify` | 0 | "Self-purification" plugin bundling a security audit, CLAUDE.md optimization, and session-pattern analysis |
| `Goodsmileduck/claude-registry` | 1 | Community plugin marketplace (DevOps skill packs, diagramming); CLAUDE.md optimization is one listed capability among several |

---

## Adjacent tools (different problem, same file)

Not direct competitors — but each exposes at least one real gap worth naming.

| Repo | Stars (2026-07-09) | What it actually does | Gap it exposes |
|---|---|---|---|
| `alirezarezvani/ClaudeForge` | 402 | **Generates** and continuously **syncs** CLAUDE.md to match the codebase; a hard 150-line cap enforced by a `PostToolUse` hook, not LLM judgment | TidyClaudeMD's own companion hook is advisory-only (it nags, never blocks). ClaudeForge shows a hard-enforced cap is a real, shipped alternative — TidyClaudeMD has no equivalent enforcement mechanism at all, hook-based or otherwise. |
| `severity1/claude-code-auto-memory` | 152 | Auto-updates CLAUDE.md at end of turn via marker-based sections, in an isolated agent to protect main-session context | TidyClaudeMD's `--memory`/`--skills` classes run inline in the invoking session, spending its context directly — no isolated-agent execution mode exists to keep a heavy tidy run's context cost off the main session. |
| `BayramAnnakov/claude-reflect` | 1,235 | Mines session history for repeatable workflow patterns to promote into slash commands, on top of capturing corrections into CLAUDE.md | `claudemd-tidy-reflect`'s only evidence source is its own structured run logs — it has no mechanism to mine raw session history for signal the way claude-reflect does. |
| `agent-sh/agnix` | 334 | Linter/LSP validating CLAUDE.md, AGENTS.md, SKILL.md, hooks, and MCP configs against 432 correctness rules; IDE plugins (VS Code, JetBrains, Neovim, Zed) + GitHub Action + autofix | agnix ships IDE-level integration (VS Code, JetBrains, Neovim, Zed) and a GitHub Action. TidyClaudeMD has no IDE presence and no CI distribution of any kind. |

---

## Gaps to close, ranked by evidence (updated 2026-07-09)

Each gap below is cited to the specific project(s) that exposed it. Ranked by how many independent projects converge on the same gap, then by how directly it's evidenced against this repo specifically.

1. **No skill behavioral testing.** *(pulser)* — `--skills` only checks structure, never behavior; no regression detection across runs despite the run-log history already having the raw material for it.
2. **No cross-skill overlap/conflict detection.** *(pulser, claudoctor — two independent projects)* — each `SKILL.md` is judged in isolation; nothing catches trigger-description overlap or near-duplicate skill bodies within the same repo.
3. **No headless / CI-native entry point.** *(geuneda, tsalkin, pulser, claudoctor, agnix — five independent projects)* — TidyClaudeMD is 100% conversational; not even `--report` mode's mechanical-checks tier, despite being designed as judgment-free, can run outside a live session.
4. **No coverage beyond Claude Code's own ecosystem.** *(wrsmith108, NicoAcosta, claudoctor — three independent projects)* — no AGENTS.md content audit (import-detection only), no copilot-instructions.md, no `~/.codex/`, `~/.hermes/`, or `~/.cursor/rules/` awareness.
5. **No exact tokenization.** *(claudoctor)* — every size/cost figure the suite reports (session-cost estimate, non-English overhead) is a heuristic, never a real tokenizer call.
6. **No sweep of plugin/marketplace caches.** *(claudoctor)* — confirmed on this exact machine: two stale copies of TidyClaudeMD itself sat undetected in `~/.claude/plugins/cache/`.
7. **No non-git reversibility fallback.** *(pulser)* — an ungated, non-git-tracked target always falls back to confirm-first; no lighter backup-and-undo tier exists in between.
8. **RELOCATE's rich-abstract pointer is unverified.** *(wrsmith108)* — the pattern is mandated by rule but never measured against TidyClaudeMD's own output the way wrsmith108 measured its own.
9. **No shareable report artifact.** *(claudoctor)* — the only durable output is the machine-oriented run log; nothing renders a human-shareable document for team review.
10. **No missing-content / completeness check.** *(claudoctor)* — TidyClaudeMD only ever asks whether existing content should stay, move, or go, never whether something important is absent.
11. **No explicit contradiction-pattern library.** *(Pinkers01)* — Consistent? probes contradiction shapes generically; no maintained list of specific, named recurring conflicts.
12. **No hard-enforced size cap.** *(ClaudeForge)* — the companion hook is advisory-only; no hook-based hard block exists as an option.
13. **No isolated-agent execution mode.** *(severity1/claude-code-auto-memory)* — heavy tidy runs spend the invoking session's own context directly.
14. **No zero-install, paste-and-check entry point.** *(Pinkers01)* — every use of TidyClaudeMD requires the plugin installed and a live session; no lightweight alternative for a one-off check or a non-Claude-Code user.

### What this doesn't change

Naming is still clear — no collision found under `TidyClaudeMD`, `claude-md-tidy`, or close variants, across either the 2026-07-03 or 2026-07-09 passes.
