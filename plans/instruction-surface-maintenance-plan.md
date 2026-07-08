# TidyClaudeMD mission expansion: maintain the whole instruction surface

Status: **proposed, not started.** Created 2026-07-08 from a user-directed reframing: the suite's mission grows from "tidy CLAUDE.md" to **maintaining every prose instruction file Claude Code loads** — keeping them slim and tidy, removing unnecessary content, relocating misplaced content, and compacting/optimizing wording. Human-in-the-loop only for decisions **only the user can make**; everything else autonomous, using iterative refinement loops.

Relationship to `plans/claude-md-scope-tensions-plan.md`: that plan stays narrow (the "where does this line belong" taxonomy for CLAUDE.md and its satellites) and ships first — its items 2–5 are the placement machinery this mission reuses. This plan defines the mission around it: targets, autonomy model, versioning, and the roadmap. Where the two touch (memory writes, confirmation flow), this plan's decisions supersede and are cross-referenced there.

## Mission

For every target file class below:

1. **Slim** — keep each file within its class's size guidance; every line earns its context cost.
2. **Remove** — unnecessary, stale, or superseded content goes (under the existing evidence rules: verified duplicates/dead references only; no-loss otherwise).
3. **Relocate** — content in the wrong scope moves to its right home (the scope-tensions plan's five axes are the taxonomy).
4. **Compact** — optimize wording of what stays: imperative bullets, tables over prose, one example max, no redundancy.

## Target classes

| Class | Location(s) | Load behavior | Hygiene focus | Status |
|---|---|---|---|---|
| Project CLAUDE.md (+ nested, `CLAUDE.local.md`) | `./CLAUDE.md`, `./.claude/CLAUDE.md`, nested, `./CLAUDE.local.md` | Full content, every session (nested: on demand) | ~150-line guardrail, five axes, seven Step 2b tests (five today; items 2–3 of the scope-tensions plan add the other two) | **Current core** — Phase 0 completes its test set |
| Global CLAUDE.md | `~/.claude/CLAUDE.md` | Full content, every session, every project | Same tests; highest blast radius per line | In scope (user-level tier) |
| Project rules | `.claude/rules/*.md` | Unconditional: at launch, CLAUDE.md-equal priority; path-scoped: when matching files are read | Right `paths:` scoping; no overlap with CLAUDE.md; one topic per file | In scope (scope-tensions item 5 is the entry point) |
| User rules | `~/.claude/rules/*.md` | At launch, before project rules, every project | Same as project rules + "is this really universal?" | In scope (user-level tier) |
| Skills | `.claude/skills/*/SKILL.md`, `~/.claude/skills/*/SKILL.md` | Description every session; body on invocation | Official "under 500 lines" guidance; description quality (it's the always-loaded part); progressive disclosure (bulk → lazily-read supporting files) | In scope — except the suite's own two SKILL.md files: self-changes belong to the reflect skill's evidence-gated loop, never to a general tidy run |
| Auto memory | `~/.claude/projects/<slug>/memory/` — `MEMORY.md` + topic files | Index: first 200 lines / 25KB every session; topic files on demand | Index stays under its cap and stays an *index*; topic files deduplicated, stale facts pruned; misplaced deliberate rules flagged for promotion | In scope — **full read+write target** (user decision 2026-07-08; supersedes the scope-tensions plan's item-3 never-write rationale at mission level) |
| Plugin-installed skills | `~/.claude/plugins/...` | As skills | — | **Excluded**: overwritten by plugin updates; improvements belong upstream in the plugin's repo |
| Slash commands, subagent defs, AGENTS.md-as-target | `.claude/commands/`, `.claude/agents/`, `AGENTS.md` | Various | — | **Deferred** — not selected in the 2026-07-08 scoping decision; revisit after the classes above work. (AGENTS.md *import detection* is already in scope-tensions item 4; deferred here is only tidying its content.) |

Per-class hygiene tests beyond the shared ones are deliberately thin at this stage — each class gets its detailed test set when its phase lands (see roadmap), evidenced by real runs, not invented up front.

## Autonomy model — reversibility-gated (user decision, 2026-07-08)

Human-in-the-loop is reserved for decisions **only the user can make**. Everything else the skill resolves itself, searching for the best solution with iterative refinement.

| Tier | Condition | Behavior |
|---|---|---|
| **Autonomous** | The target file is git-tracked (or otherwise snapshot-restorable — see memory backups below) | Analyze → apply → adversarial self-review → fix → re-review until clean; **commit per iteration** so every step is individually revertible |
| **Confirm-first** | The target file is unversioned and not snapshot-protected | Present the plan for that file, wait for approval (today's Step 4 behavior), then apply autonomously |
| **Always ask** | User-only decisions, any tier | Intent/deliberateness of a rule ("is this pinned to this repo on purpose?"), personal-preference tradeoffs, scope ownership (promote to global?), publishing/visibility, anything the PRIMARY CHECK flags |

The **iterative refinement loop** is the same shape as the global debugging-loop rule: after applying, run a fresh, skeptical review pass (an independent subagent is ideal — it isn't anchored on the edit's own reasoning) trying to find information loss, broken references, or degraded wording; fix and re-review until a pass finds nothing; commit each iteration, never one batch commit.

**Invariant 1 must be rewritten** (currently: "no CLAUDE.md is edited before the user approves the plan"). Direction approved by the user 2026-07-08; exact wording at implementation, but the shape is: *"no unversioned, non-snapshot-protected instruction file is edited before the user approves; versioned targets are edited autonomously with per-iteration commits; user-only decisions always stop."* This is a MAJOR workflow-contract change by the suite's own table — it already has the required user approval in principle, but the final invariant text is confirmed with the user once more at implementation time, since invariant edits are never applied silently.

**What "user-only" means in practice** (the lean-CHALLENGE principle): a question goes to the user only when the answer depends on their intent, preference, or knowledge the repo/docs can't contain. "Is this reference dead?" is grep-able — autonomous. "Does the team want this rule?" is not — ask. The scope-tensions plan's CHALLENGE routing already follows this line; the mission generalizes it: **CHALLENGE shrinks to genuinely user-only questions, everything evidence-resolvable is resolved in the loop.**

## Versioning `~/.claude` — evaluation (user-requested, 2026-07-08)

User-level files (`~/.claude/CLAUDE.md`, `~/.claude/rules/`, `~/.claude/skills/`) are the highest-leverage targets — they load in every session of every project — and today they're unversioned, which would trap them in the confirm-first tier forever. Versioning them makes them autonomous-tier and gives every edit a diff and a revert path. Constraints any option must respect: `~/.claude` also holds **credentials** (`.credentials.json` where the OS keychain isn't used), **auto memory and transcripts** (personal data), and **nested git repos** (`plugins/marketplaces/*` are clones) — none of which belong in version control, and nothing from this directory may ever reach a public remote (PRIMARY CHECK).

| Option | How | Pros | Cons |
|---|---|---|---|
| **1. In-place git repo with ignore-all + allowlist** (recommended) | `git init` in `~/.claude`; `.gitignore` starts with `*` and un-ignores only the instruction files — CLAUDE.md, `rules/`, `skills/`, `commands/` (commands tracked for safety even though tidying them is deferred). Exact negation patterns are an implementation detail verified at bootstrap (naive `!rules/` alone doesn't re-include a directory's contents under an ignore-all). Local-first, optional **private** remote for backup/multi-machine | Edits happen where Claude Code reads them — no sync layer; the reversibility gate reads `git ls-files` directly; ignore-all default means new sensitive files are excluded *by default*, not by remembering to exclude them; nested plugin repos are ignored automatically | A repo inside `$HOME` subtree (harmless but unusual); if a remote is ever added it **must** be private and verified so on every push (PRIMARY CHECK); allowlist needs a preflight audit so a future un-ignored path can't silently start tracking something sensitive |
| 2. Dotfiles manager (chezmoi/stow) | Files live in a dotfiles repo, applied/symlinked into `~/.claude` | Fits an existing dotfiles habit; one repo for all machine config; mature tooling | Claude Code *writes* to these files at runtime (memory compaction, skill edits, this suite itself) — two-way drift between source-of-truth and applied copy is a constant reconciliation chore; the tidy skill would have to know about the manager to commit correctly; symlinked instruction files add a failure mode the docs only recently fixed for rules |
| 3. Mirror/sync repo | A separate private repo; the skill copies files in before editing and back out after | Keeps `$HOME` clean of repos | Copies drift the moment anything else edits `~/.claude`; the "which copy is true?" question reintroduces exactly the staleness problem this suite exists to kill |
| 4. No versioning | — | Zero setup | User-level targets stay confirm-first forever; no diffs, no history, no revert — the weakest fit for an autonomous-iteration model |

**Recommendation: option 1.** It is the only option where the file Claude Code reads, the file the skill edits, and the file under version control are the same file. Bootstrap belongs in the suite as a new preflight capability: detect unversioned `~/.claude`, propose the init + allowlist `.gitignore` (shown in full, since it's the safety mechanism), create it only on the user's explicit approval — repo-initialization in the home directory is a user-only decision.

**Memory is deliberately NOT git-tracked** under option 1, even though it's a full maintenance target: it churns constantly (Claude writes it mid-session), it's machine-local by design, and it's the most personal artifact class. Instead, memory edits get their reversibility from **per-run snapshots**: before a tidy touches any memory file, copy the whole `memory/` directory to `${CLAUDE_PLUGIN_DATA}/memory-backups/<project-slug>/<date>/` (plugin data survives plugin updates and never leaves the machine). A snapshot-protected memory run qualifies for the autonomous tier; the snapshot path is stated in the run report, and restoring is a plain copy back.

## Roadmap

Phases are ordered so each one's machinery exists before the phase that needs it. Versions are indicative; the reflect skill's normal bump rules apply.

- **Phase 0 — land the scope-tensions plan** (items 2–5; item 1 rides along). The placement taxonomy (correctly-scoped, rule-vs-memory, import-following, rules destination) is the relocation engine every later phase reuses. MINOR bumps, already specced there.
- **Phase 1 — `~/.claude` versioning bootstrap.** New preflight capability: detect, propose, create the allowlist repo on approval (option 1 above). Prerequisite for putting user-level targets in the autonomous tier. MINOR.
- **Phase 2 — generalize targets.** Step 2's target-finding becomes a target-class registry (each class: locations, load behavior, size guidance, class-specific tests). First new classes: user-level CLAUDE.md/rules (they reuse the existing tests nearly verbatim), then skills (500-line guidance, description-quality test, progressive-disclosure test: "should this SKILL.md section be a lazily-read supporting file?"), then memory (index-is-an-index test, topic-file dedup, stale-fact pruning, promotion flags — plus the snapshot mechanism). Each class lands as its own MINOR release so run records accumulate per class.
- **Phase 3 — autonomy rework.** Invariant 1 rewrite (MAJOR, final wording confirmed with the user), the reversibility gate, the iterative self-review loop, lean CHALLENGE. Deliberately *after* Phase 2 has produced real multi-class run records — the autonomy change is the riskiest step and should be informed by evidence of where the skill's judgment was actually right and wrong.
- **Phase 4 — reflect-skill alignment.** Run records gain a target-class field; the reflect skill's signals and the pruning loop work per class; README/invariants updated to describe the mission. PATCH/MINOR.

## What this plan does not decide

- **Final invariant-1 wording** — confirmed with the user at Phase 3, per the invariant-gate rule.
- **Per-class detailed test sets** for skills and memory — drafted in their Phase 2 slices, evidenced by dry-runs against real instances (e.g. this suite's own SKILL.md files, this user's real memory directory) rather than invented here.
- **Whether the deferred classes** (slash commands, subagent definitions, AGENTS.md content) **join later** — revisit once Phases 1–2 have run records.
- **Multi-machine story** for option 1's private remote (sync cadence, conflict handling) — irrelevant until a second machine exists in practice.
- **Whether `--report` mode grows a whole-surface summary** ("your instruction surface: N files, M lines always-loaded, cost estimate") — attractive, cheap, but belongs with Phase 2's registry, not before.

## Iteration log

- **2026-07-08 — created.** User-directed mission expansion (conversation, 2026-07-08): all basic Claude Code instruction files as maintenance targets; HITL only for user-only decisions; iterative refinement loops. Scoping decisions same day: user-level variants in scope **with versioning** (evaluation above); slash commands / subagent defs / AGENTS.md-content deferred; mission documented as this separate plan; autonomy reversibility-gated; memory a full read+write target. Versioning decided the same day: **option 1, in-place allowlist git repo** (user decision, 2026-07-08). Self-review before first commit fixed: the CLAUDE.md row overclaiming its test set as already covered; the Skills class not excluding the suite's own SKILL.md files (reflect-skill territory); the allowlist sketch implying naive gitignore negation patterns suffice.
