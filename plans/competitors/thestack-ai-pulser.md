# TheStack-ai/pulser

**URL:** https://github.com/TheStack-ai/pulser
**Stars (2026-07-09):** 18
**Category:** Direct competitor — `--skills` class only
**License:** MIT
**Distribution:** `npm install -g pulser-cli`; registers itself as a Claude Code skill on install; also a GitHub Action (`pulser-claude-code-skill-linter`, listed on GitHub Marketplace)

## What it is

A dedicated `SKILL.md` linter — "Diagnose. Prescribe. Fix." — for Claude Code skills specifically. No CLAUDE.md, rules, or memory coverage at all; entirely scoped to the skills artifact class TidyClaudeMD added in v0.15.0.

## What it does

Scans `SKILL.md` files against 8 diagnostic rules derived directly from Anthropic's own published skills guidance ("Building Claude Code: How We Use Skills"):

| Rule | What it checks |
|---|---|
| `frontmatter` | Required `name` and `description` fields present |
| `description` | Trigger keywords, "Use when" pattern, length |
| `file-size` | SKILL.md under 500 lines |
| `gotchas` | A "Gotchas" section documenting known failure patterns exists |
| `allowed-tools` | Tool restrictions appropriate for the skill's type |
| `structure` | Supporting files present for large skills (progressive disclosure) |
| `conflicts` | Trigger-keyword overlap **between** skills in the same install |
| `usage-hooks` | A skill-usage-logging hook is installed |

Each skill is auto-classified into a type (analysis, research, generation, execution, reference) with a confidence score, and every prescription is tailored to the detected type rather than generic.

**Eval — the standout feature.** `pulser eval` actually *runs* a skill's behavior against real inputs and asserts on the output, via `claude -p`:

```yaml
tests:
  - name: "catches bugs"
    input: "Review: function add(a,b) { return a - b }"
    assert:
      - starts-with: "The bug"
      - contains: "subtract"
      - min-length: 30
```

Supported assertions: `contains`, `not-contains`, `starts-with`, `ends-with`, `min-length`, `max-length`, `matches` (regex). Runs are tracked over time and flagged when a *previously passing* test starts failing (exit code `3`, distinct from a fresh failure's exit code `1`) — a regression detector, not just a point-in-time check.

**Reversibility, the pulser way.** `--fix` auto-applies safe structural fixes with a full backup; `pulser undo` is an instant single-level rollback. Simpler than TidyClaudeMD's git-commit-per-iteration model (no history beyond one step, no revertibility if you keep applying fixes), but it means pulser works identically in a repo that isn't git-tracked at all — TidyClaudeMD's reversibility gate specifically routes ungated files to confirm-first *because* it has no non-git fallback for autonomous apply.

**Two independent entry points.** A pure CLI (`pulser`, `pulser --fix`, `pulser eval`, `--format json`/`--format md` for CI) that needs no LLM session at all for the diagnostic pass, *and* a conversational one ("check my skills" or `/pulser`) that runs the same pipeline inside Claude Code and offers to fix interactively. The CLI path is genuinely free — no LLM call — for everything except `eval`, which calls `claude -p` per test case.

**CI-native distribution.** Ships as an actual GitHub Action on the Marketplace, with `--format json`, `--strict` (warnings become errors), and documented exit codes (`0`/`1`/`2`) — built to gate a pull request, not just to run interactively.

**Numeric health score.** A single 0-100 score per scan (`54 skills scanned · Score: 89/100`), plus a per-skill severity tier (healthy/warning/error) — a legible single number a team can track over time, which none of TidyClaudeMD's five verdicts produce.

## Gaps relative to TidyClaudeMD

- No repo-comprehension gate and no cross-artifact awareness — pulser only ever looks at the `SKILL.md` files themselves, never checks a skill's claims against the actual repo (a referenced script that no longer exists, a workflow that changed).
- No equivalent of CHALLENGE — findings are diagnosed and auto-fixed or left as a warning; there's no per-item "raise this specific ambiguity to the user as a question with evidence and options" flow.
- No self-improvement loop — pulser's 8 rules are fixed; nothing in the tool learns from real scan outcomes the way `claudemd-tidy-reflect` folds evidence-backed lessons back into `claudemd-tidy`.
- CLAUDE.md, `.claude/rules/`, and auto memory are entirely out of scope — a one-artifact-class tool by design.
