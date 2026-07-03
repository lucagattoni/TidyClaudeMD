# CLAUDE.md scope-placement tensions — research + plan

Status: **proposed, not started.** Researched 2026-07-03 against official Claude Code documentation (`code.claude.com/docs/en/`: `memory.md`, `features-overview.md`, `best-practices.md`). Triggered by a real gap found in conversation: `claudemd-tidy` has no test for "this line belongs in a different file than the one it's in" — the tidy skill's five verdicts (KEEP/COMPRESS/RELOCATE/DELETE/CHALLENGE) only reason about content *within* the current repo. This doc surveys four scope-placement axes at once, since they're the same class of problem, and proposes concrete skill changes for each.

_Updated 2026-07-03: added axis 4 (CLAUDE.md vs. AGENTS.md) per user request, researched the same way as the first three._

## The four axes

Every line in a CLAUDE.md implicitly claims to belong exactly where it sits — and, per axis 4, an entire *other file* can implicitly claim rules that CLAUDE.md never sees. Four independent questions can each falsify that:

1. **CLAUDE.md vs. Skill** — is this a fact needed every session, or a procedure/reference used only sometimes?
2. **Project CLAUDE.md vs. user/global CLAUDE.md vs. `CLAUDE.local.md`** — is this project-specific, cross-project-personal, or repo-local-personal?
3. **CLAUDE.md vs. Memory (`MEMORY.md`)** — is this a deliberate, human-authored rule, or an agent-discovered pattern?
4. **CLAUDE.md vs. AGENTS.md** — does this repo have a cross-tool instruction file Claude Code doesn't read directly, and if so, is it bridged in correctly?

`claudemd-tidy` today only reasons well about axis 1 (via RELOCATE). Axes 2, 3, and 4 are unhandled — a line can be flagged True/Live/Consistent/Actionable/non-redundant and still be filed in the wrong *scope* entirely (or, per axis 4, an entire sibling file's content can be silently invisible to every Claude Code session), and nothing currently catches that.

---

## Axis 1: CLAUDE.md vs. Skill — research

Official guidance (`features-overview.md`, `best-practices.md`, `memory.md`):

- **Size ceiling**: "Keep CLAUDE.md under 200 lines" — bloated files "cause Claude to ignore your actual instructions." (This project's own global hygiene rule already uses a stricter internal ~150-line guardrail — a deliberate, tighter-than-official convention, not a discrepancy to fix.)
- **Context-cost asymmetry**: CLAUDE.md's full content loads every session; a Skill's description loads every session but its full content only loads when invoked or matched to the task ("progressive disclosure"). This is the mechanical reason procedural/reference content is expensive in CLAUDE.md and cheap in a Skill.
- **Decision rule** (official): a fact Claude should know *every* session (build commands, conventions, "never do X" rules) → CLAUDE.md. A multi-step procedure, or reference material used only sometimes, or content that "only matters for one part of the codebase" → Skill.

**Gap check against the current skill:** `claudemd-tidy`'s RELOCATE verdict test is *"Procedure/step detail whose natural owner is a skill, `docs/`, or README"* — this already matches the official decision rule closely. **Axis 1 is already substantially covered.** One small, low-priority refinement: the RELOCATE test could cite the official framing directly ("only matters for one part of the codebase," "used sometimes, not every session") to sharpen borderline calls, but this is a wording tightening, not a new mechanism — folded into item 1 below as an optional COMPRESS-level polish, not a new verdict.

## Axis 2: Project vs. user/global vs. local CLAUDE.md — research

Official guidance (`memory.md`, "Memory" scope table and "How CLAUDE.md files load"):

| Scope | Location | Purpose | Shared with |
|---|---|---|---|
| User instructions | `~/.claude/CLAUDE.md` | Personal preferences for **all** projects | Just you |
| Project instructions | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Team-shared instructions | Team, via source control |
| Local instructions | `./CLAUDE.local.md` (gitignored) | Personal, **project-specific** preferences | Just you, this project only |

Key mechanics: all scopes are **additive, not overriding** — every discovered file concatenates into context (root-down by directory, `CLAUDE.local.md` appended after `CLAUDE.md` at each level). Nested per-directory CLAUDE.md files are a documented, first-class pattern for monorepos, loaded on demand when Claude reads files in that subdirectory.

**Decision rule** (official): personal, cross-project habits → user `~/.claude/CLAUDE.md`. Team-shared, project-specific conventions → project `CLAUDE.md`, git-tracked and PR-reviewed. Personal preferences specific to *this* project only (sandbox URLs, personal branch names, local test data) → `CLAUDE.local.md`, gitignored.

**Gap check against the current skill:** nothing. This axis is entirely unhandled today. A project `CLAUDE.md` line that is actually a generic, cross-project working preference (and doesn't already exist in the global file — if it did, that's Redundant-by-order, a different, already-covered case) currently has no test that would flag it. Same for a line that's a personal, repo-specific preference someone hardcoded into the shared, team-reviewed `CLAUDE.md` instead of `CLAUDE.local.md`. **This is the concrete gap the user identified in conversation** — proposed fix below (item 2).

## Axis 3: CLAUDE.md vs. Memory (`MEMORY.md`) — research

Official guidance (`memory.md`, "CLAUDE.md vs auto memory"):

| Aspect | CLAUDE.md | Auto Memory |
|---|---|---|
| Who writes it | You (human-authored) | Claude (agent-discovered) |
| Contains | Instructions and rules | Learnings and patterns |
| Loaded into context | Full content, every session | Index only (`MEMORY.md`, capped at 200 lines / 25KB) — topic files recalled on demand, not preloaded |
| Use for | Coding standards, workflows, architecture | Build commands Claude discovered, debugging insights, preferences Claude noticed |

Neither CLAUDE.md nor Memory *enforces* anything — both are context that shapes reasoning, not a hard gate (for hard guarantees, official guidance points to `PreToolUse` hooks instead). The documented **promotion path**: when something Claude learned in Memory turns out to matter every session, the user says "add this to CLAUDE.md" (or edits directly) — moving it from agent-discovered-and-ephemeral to human-decided-and-durable.

**Decision rule** (official): deliberate, stable, team-relevant rules → CLAUDE.md. Ad hoc discovered patterns, debugging insights, machine-local tool preferences → Memory, not CLAUDE.md.

**Gap check against the current skill:** also entirely unhandled. A CLAUDE.md line that reads like a discovered debugging insight or an ad hoc noticed pattern (not a deliberate team rule) rather than an intentional instruction is invisible to all five current verdicts — True/Live/Consistent/Actionable/Redundant-by-order all pass a line like this if it happens to be accurate and unambiguous, even though it's arguably the wrong *kind* of content for a PR-reviewed rulebook. Proposed fix below (item 3) — deliberately scoped narrower than the full bidirectional promotion path, for reasons explained there.

## Axis 4: CLAUDE.md vs. AGENTS.md — research

Official guidance (`memory.md`, "AGENTS.md" section):

- **Claude Code does not read `AGENTS.md` directly** — stated explicitly: *"Claude Code reads `CLAUDE.md`, not `AGENTS.md`."* No setting or flag changes this; it's a firm platform boundary, not a gap in configuration.
- **Documented interop pattern**: a `CLAUDE.md` can start with an `@AGENTS.md` import line — Claude loads the imported file at session start, then reads any remaining `CLAUDE.md` content below it as Claude-specific additions. Alternative: `ln -s AGENTS.md CLAUDE.md` (a symlink; Unix-only, needs Admin/Developer Mode on Windows).
- **`/init` behavior**: running `/init` in a repo that already has an `AGENTS.md` reads it and folds the relevant parts into the `CLAUDE.md` it generates (also reads `.cursorrules`, `.devin/rules/`, `.windsurfrules` — other tools' equivalents).

**Decision rule** (official): a repo supporting multiple AI coding tools should keep shared instructions in `AGENTS.md` and import it into `CLAUDE.md` (`@AGENTS.md` at the top, Claude-specific content below), rather than maintaining two separately-duplicated files.

**Gap check against the current skill:** two distinct gaps, both unhandled today, sharing one root cause — the skill doesn't know `AGENTS.md` exists at all:

(a) **Repo-level structural gap.** Step 2's target-finding (bullet 1) looks for `*CLAUDE.md`/`CLAUDE.md` and `CLAUDE.local.md`-style files; it never checks for `AGENTS.md`. A repo with an `AGENTS.md` that no `CLAUDE.md` imports has a real, concrete consequence beyond this skill: Claude Code itself never reads that content in *any* session, tidy or otherwise. Two sub-cases: no `CLAUDE.md` exists at all (Claude Code has zero project-specific context, even though other tools using `AGENTS.md` do), or a `CLAUDE.md` exists but doesn't import `AGENTS.md` (the blindness problem, plus a duplication/drift risk if both files independently maintain overlapping rules).

(b) **Import-following gap.** When a `CLAUDE.md` *does* correctly import `AGENTS.md` via `@AGENTS.md`, that imported content is exactly as "loaded every session" as anything written directly in `CLAUDE.md` — but Step 1's rule-loading and Step 2b's line-by-line interrogation only look at the literal `CLAUDE.md` file's own text today. An import line is opaque to the skill; the content it pulls in is never audited at all, even though it's fully active in every session.

---

## Proposed changes to `claudemd-tidy`

### 1. Axis 1 — sharpen RELOCATE's citation (low priority, optional)

**Proposed fix.** Add the official framing directly to the RELOCATE test in the Step 3 verdict table: *"...or content that only matters for one part of the codebase, or is used occasionally rather than every session (official Claude Code guidance: CLAUDE.md is for facts needed every session; Skills are for progressive-disclosure reference/procedure content)."* This doesn't change behavior — RELOCATE already catches these cases — it just gives the model a citable, sharper rule of thumb for borderline calls between "leave as COMPRESS" and "RELOCATE to a skill."

**Files:** `skills/claudemd-tidy/SKILL.md` (Step 3 verdict table, RELOCATE row).
**Bump:** PATCH (wording sharpened, no new capability).

### 2. Axis 2 — new Step 2b test: **Correctly-scoped?**

**Proposed fix.** Add a sixth Step 2b question: *"Correctly-scoped? Is this line's content actually about **this repo** — or is it a generic, cross-project working preference (→ belongs in user `~/.claude/CLAUDE.md`) or a personal-but-repo-specific preference not relevant to the rest of the team (→ belongs in `CLAUDE.local.md`, gitignored, create if missing)?"*

Routing: failing this test alone (the line is otherwise True/Live/Consistent/Actionable/non-redundant) is not "wrong," just misplaced — route to **CHALLENGE**, never straight to RELOCATE or DELETE, since only the user can confirm whether something is genuinely meant to be universal/personal versus deliberately pinned to this one project. Extend CHALLENGE's existing option set (`fix to match reality / keep — I'm missing context / delete`) with two more: *promote to global `~/.claude/CLAUDE.md`* / *move to `CLAUDE.local.md`*.

**Apply-phase consideration — this is the one genuinely new kind of write the skill would perform.** Every existing RELOCATE destination (`.claude/commands/`, `.claude/skills/`, `docs/`, `plans/`, `README.md`) lives *inside* the repo being tidied. "Promote to global `~/.claude/CLAUDE.md`" writes *outside* the repo, into a file shared across every project the user has. This is the same class of action `claudemd-tidy-reflect` already treats with extra care (Step 3: *"Never fork a private copy of those rules into the skill; propose the global edit and get explicit user confirmation, since that rulebook governs every session"*) — the same discipline should apply here: a promote-to-global resolution should be called out individually at confirmation time, not silently bundled into a bulk "approve all" the way an in-repo RELOCATE can be.

**Files:** `skills/claudemd-tidy/SKILL.md` (Step 2b new question, Step 3 CHALLENGE option list, Step 4 confirmation flow, Step 5 apply-phase — new destination case).
**Bump:** MINOR (new test, extends CHALLENGE's option set — not a new verdict, so not MAJOR by the suite's own table).

### 3. Axis 3 — new Step 2b test: **Rule-vs-memory?** (one direction only, deliberately)

**Proposed fix.** Add a seventh Step 2b question: *"Rule-vs-memory? Does this line read as a deliberate, team-decided instruction — or as a discovered pattern, debugging insight, or ad hoc noticed preference that reads more like something Claude learned than something the team decided?"*

Routing: same as Correctly-scoped? — failing this test alone routes to **CHALLENGE**, with the question surfaced as *"this reads like a discovered pattern rather than a deliberate rule — keep it here, or would this fit better in your memory system instead?"* **Deliberately no auto-apply and no attempt to write into `~/.claude/projects/<slug>/memory/` on the user's behalf.** Two reasons to stop at CHALLENGE and go no further:

- Memory files are explicitly documented as accumulating content that "routinely holds personal data" (this project's own global PRIMARY CHECK rule already says so) — writing into that location autonomously, even on request, adds a privacy-surface the skill doesn't currently touch anywhere else.
- The reverse direction — reading Memory to find content that should be *promoted* to CLAUDE.md — would require `claudemd-tidy` to read a completely new artifact class it has no relationship with today (Step 2's survey never touches Memory), and that promotion path is already officially documented as a manual, user-initiated action ("ask Claude to add this to CLAUDE.md, or edit the file yourself"). Not worth building a redundant path for something the platform already handles.

This keeps the check useful (flags miscategorized content) without expanding the skill's read/write surface into a new, more sensitive artifact type.

**Files:** `skills/claudemd-tidy/SKILL.md` (Step 2b new question, Step 3 CHALLENGE option list).
**Bump:** MINOR (new test, extends CHALLENGE — not a new verdict).

### 4. Axis 4 — detect `AGENTS.md` and follow `@AGENTS.md` imports

**Proposed fix (two parts, one item — both stem from the same root cause: the skill doesn't know `AGENTS.md` exists).**

**Part a — target-finding.** Extend Step 2 bullet 1 to also check for `AGENTS.md` at the repo root (and nested, mirroring how `CLAUDE.md` is found). If found, check whether any `CLAUDE.md` in the repo imports it (`@AGENTS.md` line, or a symlink). Surface the result in the Step 4 plan as an explicit repo-level note, the same treatment already given to the `CLAUDE.local.md` finding: *"Found AGENTS.md at `<path>` — [not imported by any CLAUDE.md / imported by `<file>`]. Claude Code only reads CLAUDE.md directly; an unimported AGENTS.md is invisible to every Claude Code session, not just this tidy run."* Never auto-adds an import line (that's a decision the user should make deliberately) — always surfaces as information at minimum, and escalates to a CHALLENGE-style question when it looks like unintentional drift (both files exist, neither imports the other, and a grep shows overlapping rule text between them). Cheap enough to also run during `--report` mode, alongside the existing mechanical checks.

**Part b — follow imports during interrogation.** When a target `CLAUDE.md`'s content is (or starts with) `@AGENTS.md` — or the file is a symlink to an `AGENTS.md` — treat the imported file's content as part of the "effective" CLAUDE.md for every subsequent step: Step 2b's line-by-line walk includes `AGENTS.md`'s lines, Step 3's verdicts can apply to blocks that physically live in `AGENTS.md`, and Step 5's apply phase edits `AGENTS.md` directly for any verdict landing on an imported block (never duplicating imported content back into `CLAUDE.md`). This makes the official import pattern fully transparent to the skill instead of a blind spot.

**Files:** `skills/claudemd-tidy/SKILL.md` (Step 2 target-finding, Step 1/Step 2b import-following, Step 3/Step 5 — verdicts and edits can target `AGENTS.md` when imported).
**Bump:** MINOR (new detection + import-following capability, extends existing steps — no new verdict, no workflow-contract change).

---

## What this plan does not decide

- **Whether item 2's promote-to-global write needs to be an explicit seventh protected invariant**, or just documented apply-phase discipline (a note in Step 5, no invariant-gate escalation). The reflect skill's existing invariants list is specifically about *self-improvement* changing the *skill's own* behavior; this is the *tidy skill's normal operation* writing to a file outside the repo, a different category. Worth an explicit decision before implementing item 2, not assumed here.
- **Naming**: "Correctly-scoped?", "Rule-vs-memory?" are working names, not final — Step 2b's existing five questions (True/Live/Consistent/Actionable/Redundant-by-order) all read as single adjectives or a compact compound; these are a bit longer. Fine to bikeshed at implementation time, not a blocker.
- Whether item 1 is worth doing at all, given it's flagged as already-substantially-covered — included for completeness since the research surfaced it, not because it's clearly worth a release on its own.
- **Item 4's exact CHALLENGE-escalation threshold** ("looks like unintentional drift") isn't fully specified — "overlapping rule text" is doing a lot of work in that sentence and needs a concrete evidence rule (grep-based? semantic similarity? both files mentioning the same tool/command by name?) analogous to how DELETE's evidence rule was made concrete (cited duplicate location or grep-confirmed dead reference). Resolve at implementation time, don't hand-wave it into the shipped skill.

## Suggested execution order

Item 2 (Axis 2) is the one the user explicitly asked for first and the most concrete gap — do it first. Item 3 (Axis 3) is a natural companion, same shape of fix, no new invariant question attached to it (it never writes anywhere, CHALLENGE-only) — safe to do in the same pass as item 2 once the open question above is resolved. Item 4 (Axis 4, added on request) is independent of items 2/3 — different files, different mechanism (target-finding + import-following, not a Step 2b line test) — and part (a) alone (detection + reporting, no import-following) is a smaller, self-contained slice worth shipping on its own if part (b) needs more design work. Item 1 is optional polish, doable anytime, independent of everything else.
