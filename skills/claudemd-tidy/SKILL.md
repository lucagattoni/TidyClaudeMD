---
name: claudemd-tidy
description: Audit and slim Claude Code instruction files against the global "CLAUDE.md hygiene" rules — project and user-level CLAUDE.md, .claude/rules, SKILL.md files (--skills), and auto memory (--memory) — by relocating/compressing content, never losing information. Use when the user asks to tidy, slim, audit, or clean up a CLAUDE.md, their skills, rules, or memory.
version: 0.21.0
---

# /tidyclaudemd:claudemd-tidy

Audit and slim Claude Code instruction files — project CLAUDE.md by default; user-level files, skills, and memory via the class flags below. Two phases: **analyze & propose** (always), then **apply** — autonomously for reversibility-protected targets, after confirmation for everything else (the gate in Step 4).

## Argument

Optional: a path to a specific target file, a target-class flag (below), and/or `--report` to run report mode instead of a full tidy. With no flag and no path, process every CLAUDE.md in the current repo (root + nested). `--all` runs every class in scope on this machine.

## Target classes

Every class runs the same phases — preflight → rules → survey → interrogate → verdicts → confirm → apply → record — with the class-specific behavior below. For user-level classes, "the repo" in Step 0 is `~/.claude` itself: the versioning check (Step 0.7) decides whether it has git history; there is no CI/encryption surface; PRIMARY CHECK applies if the `~/.claude` repo ever has a remote. For `--memory`, Step 0's git steps don't apply at all — the snapshot rule in the table below is the safety mechanism. `--report` composes with any class flag: the mechanical checks run against that class's files. The AGENTS.md visibility check (Step 2 bullet 1 / report mode) applies to **project-class targets only** — user-level, skills, and memory classes have no AGENTS.md-equivalent surface to check.

| Class | Flag | Locations | Size guidance | Class-specific behavior |
|---|---|---|---|---|
| Project CLAUDE.md | (default) | `./CLAUDE.md`, `./.claude/CLAUDE.md`, nested, `CLAUDE.local.md` | ~150-line guardrail | The seven Step 2b questions as written |
| User-level | `--user` | `~/.claude/CLAUDE.md`, `~/.claude/rules/*.md` | same guardrail per file | **Correctly-scoped? inverts**: "is this genuinely universal — or does it belong in one project's CLAUDE.md, or as a path-scoped rule?" Earlier-loaded scopes for Redundant-by-order: managed policy (for the global CLAUDE.md); managed policy + the global CLAUDE.md (for user rules). Everything here is personal by definition — CHALLENGE still fires for intent questions, but there is no "team" dimension |
| Skills | `--skills` | `.claude/skills/*/SKILL.md`; with `--user` also `~/.claude/skills/*/SKILL.md`. **Never this suite's own two skills** — their changes belong exclusively to the evidence-gated reflect loop | official "under 500 lines" per SKILL.md | Three per-file tests: **Description-quality?** — the description is the skill's always-loaded part; does it state what the skill does *and* when to trigger it, concretely enough to fire at the right moments? **Progressive-disclosure?** — reference/example bulk not needed on every invocation belongs in a supporting file linked from SKILL.md (read lazily), not inline; that is this class's RELOCATE destination. **Frontmatter-sane?** — valid YAML, `name:` matches the directory, files the skill references actually exist (grep-confirmable → normal DELETE/fix evidence rules). **Sweep-level checks** — run once across every skill found, not per-file: **Duplicate?** (byte-identical `SKILL.md` content at ≥2 paths — hash each file, group by hash); **Near-duplicate?** (identical body but different frontmatter — hash content with the YAML frontmatter block stripped, group, excluding groups whose full-content hash also matches, since those are Duplicate?); **Name-conflict?** (same frontmatter `name:` in ≥2 skills whose content differs, restricted to the two shapes Claude Code doesn't already disambiguate on its own: (a) a `name:` differing from its directory basename that collides with another skill's name — the mismatch itself is Frontmatter-sane?'s job, this flags who it collides with — or (b) the same name reused across project and user scope, where load precedence silently picks a winner); **Trigger-overlap?** (a mechanical pre-filter — quoted phrases shared across ≥2 descriptions, plus high word overlap — surfaces candidate pairs, then judge directly whether the two descriptions would plausibly fire on the same request). A skill with malformed frontmatter (no parseable `name:`/`description:`) is skipped by all four sweep-level checks — Frontmatter-sane? already owns flagging it. All four route to **CHALLENGE**, never an auto-DELETE of a whole skill (removing one is a scope/intent decision, not a dedup line-edit): Duplicate?/Near-duplicate? cite both paths and the matching hash; Name-conflict? is tier-1 stakes (it affects which skill actually runs when invoked); Trigger-overlap? is minor tier unless one side is safety-relevant. The suite's own two skills participate as **comparison references only** — read from `${CLAUDE_PLUGIN_ROOT}/skills/`, never hashed as a removable copy — so a third-party skill duplicating one of them is flagged, with the finding landing on the third-party copy |
| Memory | `--memory` | `~/.claude/projects/<slug>/memory/` for the current project: `MEMORY.md` + topic files | index: first 200 lines / 25KB loaded per session | **Snapshot first — no snapshot, no edit**: before any change, copy the whole `memory/` directory to `${CLAUDE_PLUGIN_DATA}/memory-backups/<slug>/<YYYY-MM-DD-HHMM>/` and state the path in the plan and the report (restore = copy back; memory is never git-tracked). Additional tests: **Index-is-an-index?** — `MEMORY.md` within its cap, one line per memory, details in topic files; **Topic-dedup?** — the same fact in two files is a normal cited-duplicate DELETE; **Still-true?** — stale discovered facts pruned under the same grep/verification evidence rules; **Promote?** — an entry reading like a deliberate, every-session rule → CHALLENGE suggesting promotion to CLAUDE.md (the reverse of Rule-vs-memory?; the user decides, and the promotion itself follows Step 4's out-of-repo/individual-confirmation rule). Memory content must never land in any repo-tracked file — it is personal by default (PRIMARY CHECK, doubly so for public repos) |

## Report mode (`--report`)

A cheap, no-plan, no-confirmation, no-edits pre-check for "how bad is this CLAUDE.md right now?" — skips Step 2's survey and Step 2b's seven-test interrogation entirely. Runs Step 0 (Preflight) as normal, then reports two clearly-separated tiers of output and stops:

- **Mechanical checks** (scriptable, no judgment): line/token count per file vs. the global hygiene guardrail (~150 lines); non-English/CJK content flagged with an estimated token-overhead cost (non-English content runs roughly 30-50% more tokens per instruction than equivalent English); a session-cost estimate (per-request token cost × an assumed 30-turn session); an **AGENTS.md visibility check** (present in the repo? imported by any CLAUDE.md, or invisible to every Claude Code session?); composed with `--skills`, also the three mechanical sweep-level counts — Duplicate?/Near-duplicate?/Name-conflict? (Target classes table, Skills row) — Trigger-overlap? is judgment, so it's excluded from report mode.
- **A rough verdict-mix impression** from a shallow read only — an approximate KEEP/COMPRESS/RELOCATE/DELETE/CHALLENGE tally. Label this explicitly as unverified judgment in the output, never with the same confidence as the mechanical counts above — verdict classification without Step 2b's real interrogation is a guess, not an analysis.

A report-only run still writes a Step 7 run log (tag it analyze-only in the Result field) and explicitly states in its output that it is a heuristic pre-check, not a substitute for a full tidy.

## Step 0 — Preflight

1. **Version-currency check.** Read `~/.claude/plugins/installed_plugins.json` and find this plugin's pinned version (`plugins["tidyclaudemd@tidyclaudemd"][0].version`). Compare it to this file's own frontmatter `version:`. If they differ, this session is executing stale, cached skill instructions — **stop before Step 1** and tell the user: run `/plugin update tidyclaudemd` if not already done, then **restart the Claude Code session** (`/reload-plugins` mid-session updates the plugin registry but does not swap in new skill content for the running session — confirmed by direct observation, 2026-07-08). If the two versions match, proceed normally.
2. Confirm you are inside a git repo; if not, ask which file to tidy and skip git steps.
3. Sync per the global working defaults: `git fetch`, check branch/ahead-behind/dirty state.
4. Check repo visibility (`gh repo view --json visibility` or inspect the remote). If public or unknown → the **PRIMARY CHECK** in `~/.claude/CLAUDE.md` applies to every file this skill writes, including relocated content.
5. **Encryption check.** Scan for git-crypt/SOPS filters in `.gitattributes` or an `.age` key reference. If found, flag it: in Step 3, any encryption-unlock instructions in the CLAUDE.md are force-classified **KEEP** (never RELOCATE — moving them risks a chicken-and-egg lock-out); any other RELOCATE destination must be verified as covered by the same encryption scope as the source before Step 5 executes it.
6. **CI-dependency check.** Scan CI config (`.github/workflows/` and any other CI directories at the repo root) for scripts that reference `CLAUDE.md` — or any file a CLAUDE.md imports — by filename (e.g. a script that greps its content for a required phrase). If found, flag the content those scripts appear to depend on: it's ineligible for RELOCATE in Step 5 without the user's explicit confirmation in Step 4 that the CI dependency is accounted for.

7. **User-level versioning check.** If this run may write anything under `~/.claude` (a promote-to-global resolution is on the table, or a user-level target is in scope): check whether `~/.claude` is a git repository. If it isn't, **propose — never auto-create** — the bootstrap (repo-initialization in the home directory is a user-only decision): `git init` plus this allowlist `.gitignore`, shown to the user in full since it is the safety mechanism:

   ```gitignore
   *
   !.gitignore
   !CLAUDE.md
   !rules/
   !rules/**
   !skills/
   !skills/**
   !commands/
   !commands/**
   ```

   The ignore-all default keeps credentials, transcripts, auto memory, and plugins untracked *by default*, not by remembering to exclude them. Track only `*.md` and skill assets; verify with `git status` after init that nothing sensitive is staged. If the user declines, `~/.claude` writes stay confirm-first and the Step 6 report notes them as unversioned. If a remote is ever configured for this repo, it must be verified **private** before any push (PRIMARY CHECK) — memory is never tracked regardless.

## Step 1 — Load the rules

Read the **"CLAUDE.md hygiene — keep every CLAUDE.md slim"** section of `~/.claude/CLAUDE.md`. Those numbered rules are the single source of truth for this skill — do not work from memory of them.

**If the section is missing** (first-time setup): append the contents of `${CLAUDE_PLUGIN_ROOT}/HYGIENE-RULES-TEMPLATE.md` verbatim to the end of `~/.claude/CLAUDE.md` as a new section — do not overwrite or reorder anything already in that file. Tell the user you did this and why. This is a **one-time bootstrap**: from this point on, `~/.claude/CLAUDE.md` is the sole source of truth, exactly as if the section had always been there — this skill never reads the bundled template again after the first install, and the reflect skill only ever proposes edits to the global file, never to the template.

Also read the **rest** of `~/.claude/CLAUDE.md` in full (every earlier-loaded scope, not just the hygiene section) — needed for Step 2b's **Redundant-by-order?** question, which can't be answered from the hygiene section alone. If the global file itself contains live `@imports`, read those too: the redundancy comparison must see the global file's *effective* content, not just its literal text. Read every other earlier-loaded scope that exists on this machine as well: `~/.claude/rules/*.md` (user rules load before all project files) and a managed-policy CLAUDE.md if present (macOS `/Library/Application Support/ClaudeCode/CLAUDE.md`, Linux/WSL `/etc/claude-code/CLAUDE.md`, or a `claudeMd` key in managed settings) — Redundant-by-order can't be answered without them. (**Project** `.claude/rules/` files load *after* the project CLAUDE.md, so they are not earlier-loaded for its lines — overlap with them is handled in Step 2b/Step 3, not here.)

## Step 2 — Build a complete picture of the repo

**Do not analyze a single CLAUDE.md line before this step is done.** Every verdict in Step 3 must be grounded in what the repo *actually* is, not in what the CLAUDE.md claims it is.

1. Find targets: `git ls-files '*CLAUDE.md' 'CLAUDE.md'` plus an untracked-file check; include nested ones. **Also** glob the filesystem directly for common gitignored personal-override filenames (starting with `CLAUDE.local.md`) — `git ls-files` and a standard untracked-file check both skip gitignored paths, so a personal-override convention would otherwise never surface. List any found explicitly in the Step 4 plan even if the user then chooses to exclude them — that must be a stated choice, not a silent gap. For each target found, record line count, word count, section list.
   **Imports:** detect every **live `@import`** in each found CLAUDE.md — an `@path` occurrence *outside* backticks and fenced code blocks (best-effort against Claude Code's documented parsing rules; a code-span mention like `` `@README` `` is literal text, never an import). An **external** import (pointing outside the repo: `@~/...` or an absolute outside path) may be permanently inert if the user once declined Claude Code's one-time approval dialog — a state invisible in file content — so hedge any finding that depends on externally-imported content ("if this import was declined, this content may not be active in-session"); in-repo imports are not approval-gated and are assumed live. (An `InstructionsLoaded` hook is the optional companion for ground truth about what actually loads.)
   **AGENTS.md:** also check for `AGENTS.md` at the repo root and nested. If found, determine whether any CLAUDE.md pulls it in — live `@import` or symlink; that is the only direction that matters (Claude Code doesn't read AGENTS.md; other tools don't parse `@import`). State the result in the Step 4 plan as a repo-level note: *"Found AGENTS.md at `<path>` — [not imported by any CLAUDE.md / imported by `<file>`]. Claude Code only reads CLAUDE.md; an unimported AGENTS.md is invisible to every Claude Code session."* Escalate to a **repo-level CHALLENGE** — a deliberate, documented extension of the otherwise per-block CHALLENGE verdict — when the drift looks unintentional: both files exist, no import/symlink bridges them, and grep finds overlapping rule text in both. Never auto-add an import line.
2. **Fixed general-orientation pass** (always, regardless of CLAUDE.md size): top-level `README.md`; a listing — not a deep read — of `.claude/commands/` and `.claude/skills/`; `.claude/settings.json`; a recursive listing of `.claude/rules/*.md`, noting which have `paths:` frontmatter (path-scoped — load only when Claude works with matching files) and which don't (unconditional — always-loaded, CLAUDE.md-equal priority; read these in full, Step 2b's Consistent? check needs their content). For `--skills` runs, also run the Skills row's sweep-level checks (Target classes table) across every skill found in this pass.
3. **Claim-driven verification**: extract every concrete claim each CLAUDE.md makes — file paths, command names, workflow descriptions, tool/skill references — then verify only those against the repo (targeted file/path lookups, package manifests or CI config only if a claim points there, git history only if a claim makes an assertion about change frequency or ownership). Do not read anything the CLAUDE.md makes no claim about; this is the survey's cost control — it scales with the CLAUDE.md's own length, not the repo's size.
4. Inventory the possible relocation homes: `.claude/commands/` and `.claude/skills/` (per-task procedures), `.claude/rules/<topic>.md` (facts/constraints scoped to part of the codebase, as a path-scoped rule — raise a question before *introducing* `.claude/rules/` to a repo that doesn't use it yet; once bullet 2's survey found the directory already present, it's an established destination — RELOCATE into it like any other, no repeat question), `docs/` or `plans/` (background/design), `README.md` (human-facing description).
5. **Comprehension gate** — before moving on, you must be able to answer: what does this repo do; how is it built/run/deployed; which agents, hooks, and scheduled processes operate on it; which conventions demonstrably hold in the tree (naming, layout, workflows) and which appear only in the CLAUDE.md. If you can't, **keep resolving unverified claims** from bullet 3 — never expand into an open-ended survey.

## Step 2b — Critically interrogate every line

Walk each **effective** CLAUDE.md **line by line** and question whether it makes sense, before any slimming verdict. "Effective" means: for every live import found in Step 2, the imported file's lines are part of this walk (recursively, to the documented maximum of four hops — bound the walk by depth, not by trust: cycle detection is not documented for imports), and Step 3 verdicts may land on blocks that physically live in an imported file. For each rule/claim ask:

- **True?** Does it match the repo picture — do the files, commands, paths, and behaviors it references exist and work as described?
- **Live?** Is it still needed, or does it govern a workflow/file that no longer exists or was superseded?
- **Consistent?** Does it contradict another rule in this file, a nested CLAUDE.md, an unconditional `.claude/rules/` file (always-loaded context, loading after the project CLAUDE.md — so contradiction/duplication with one is a consistency matter, not Redundant-by-order), a skill, or the repo's observable conventions? Actively probe for common contradiction **shapes**, not just whatever a line-by-line read happens to surface: autonomy vs. ask-first, a stated default vs. an exception elsewhere, two rules naming different values for the same threshold/setting, "always X" vs. a documented case where the repo does not-X.
- **Actionable?** Is it unambiguous enough that two sessions would follow it the same way?
- **Redundant-by-order?** Does a global-scope file (or any other earlier-loaded rule, per Claude Code's config load order: global CLAUDE.md → global rules → project CLAUDE.md → project rules → Memory) already establish this same intent, making this line pure repetition regardless of exact wording?
- **Correctly-scoped?** Is this line's content actually about **this repo** — or is it a generic, cross-project working preference (→ belongs in user `~/.claude/CLAUDE.md`) or a personal-but-repo-specific preference not relevant to the rest of the team (→ belongs in `CLAUDE.local.md`, gitignored)? This question fires only when the content is **not** already established in an earlier-loaded scope — if it is, that's Redundant-by-order (above), not a scope question.
- **Rule-vs-memory?** Does this line read as a deliberate, team-decided instruction — or as a discovered pattern, debugging insight, or ad hoc noticed preference that reads more like something Claude learned than something the team decided?

A line that fails True / Live / Consistent / Actionable is **suspect** — never silently fix, keep, or drop it; route it to the CHALLENGE verdict below. A line that fails only **Redundant-by-order** is not suspect in the same sense (it isn't wrong, just repeated) — it routes to COMPRESS or CHALLENGE per the Step 3 verdict table, never straight to DELETE. A line that fails only **Correctly-scoped?** or **Rule-vs-memory?** is misplaced, not wrong — it routes to **CHALLENGE**, never straight to RELOCATE or DELETE: only the user can confirm whether content is genuinely universal, personal, or learned versus deliberately pinned where it sits.

## Step 3 — Analyze

Walk each CLAUDE.md section by section and classify every block with exactly one verdict, using the Step 2 repo picture and the Step 2b interrogation results:

| Verdict | Meaning | Test |
|---|---|---|
| **KEEP** | Behavior-changing rule, already terse | Passes the behavior test and all Step 2b questions as-is |
| **COMPRESS** | Rule stays here, rewritten tighter | Same meaning expressible in fewer lines (prose → bullets/table, redundant examples dropped), **or** fails only **Redundant-by-order** and trimming to a one-line acknowledgment of the earlier-loaded rule preserves genuine project-specific nuance |
| **RELOCATE** | Detail moves to its proper home; a **rich-abstract pointer** stays — a short synopsis stating the concrete fact(s) a reader would otherwise need to open the sub-doc for (the specific convention, threshold, or command, not just that a sub-doc exists), plus the link | Procedure/step detail whose natural owner is a skill, `docs/`, or README; **or** a fact/constraint that only matters for one part of the codebase → a path-scoped `.claude/rules/` file (official guidance: CLAUDE.md is for facts needed every session; skills and path-scoped rules are for content used only sometimes or only for some files) |
| **DELETE** | Removed outright | Duplicated verbatim elsewhere (cite location), **or** dead and grep-*confirmed* gone (cite the failed lookup — a rename counts as confirmed if the new location is found and cited instead) |
| **CHALLENGE** | Suspect — raised to the user as a question, never resolved unilaterally | Failed a Step 2b question but **not** grep-confirmable either way: ambiguous liveness, possible-but-unconfirmed rename, contradiction between rules, or wording too vague to act on; **or** failed only Correctly-scoped? / Rule-vs-memory? (scope and intent are never grep-confirmable — always the user's call); **or** fails only Redundant-by-order with no confirmed project-specific nuance (cite the earlier-loaded rule; the user decides compress-further vs. delete — never route this straight to DELETE, which requires a verbatim duplicate or grep-confirmed dead reference, neither of which a same-intent-different-wording match is) — **or** an otherwise-RELOCATE-eligible block blocked by an unresolved Step 0 encryption/CI-dependency flag (see Rules of evidence below) |

DELETE and CHALLENGE are mutually exclusive by evidence strength, not by which Step 2b question failed: a **confirmed** dead reference is DELETE; anything the grep can't settle is CHALLENGE. If a search is inconclusive, default to CHALLENGE — never guess a DELETE.

**Lean CHALLENGE principle:** CHALLENGE is reserved for genuinely user-only questions — intent, preference, scope ownership. Anything resolvable by evidence (a grep, a doc check, the repo picture, another verification pass) must be resolved autonomously in this step, iterating until settled, never delegated to the user for convenience.

Rules of evidence:
- Before any **DELETE**, verify: grep the repo for the referenced file/command/term. Cite the evidence (the duplicate's location, or the failed lookup proving it's gone) in the plan.
- Before any **RELOCATE**, identify the exact destination file and whether it already covers the content (then it's a DELETE-as-duplicate with a pointer check, not a move).
- Content whose only sin is verbosity is COMPRESS, never DELETE.
- **Duplicates in rules files split by load scope:** a CLAUDE.md line duplicated verbatim in an **unconditional** `.claude/rules/` file is DELETE-eligible (cite it — both copies are always-loaded, removing one changes nothing semantically); duplicated verbatim in a **path-scoped** rule it routes to CHALLENGE instead — deleting the CLAUDE.md copy would narrow always-loaded content to conditional loading, a scope decision only the user can make.
- A block flagged by Step 0's encryption or CI-dependency check cannot be RELOCATEd until that flag's condition is satisfied (matching encryption scope confirmed, or the user explicitly confirms the CI dependency is handled in Step 4) — until then it routes to CHALLENGE, not RELOCATE.
- **Skills-class sweep-level findings** (Duplicate?/Near-duplicate?/Name-conflict?/Trigger-overlap? — Target classes table, Skills row) always route to CHALLENGE, tiered per that row; never auto-DELETE a whole skill file.

## Step 4 — Report, then apply the reversibility gate

Present a plan per file (contents below), then route each target through the gate:

- **Autonomous tier** — the target file is git-tracked, or it is memory protected by a Step-taken snapshot: non-CHALLENGE verdicts proceed straight to Step 5, with a commit per iteration so every change is individually revertible. Say so explicitly in the output (which tier, which protection) — autonomy is announced, never silent.
- **Confirm-first tier** — the target is unversioned and unprotected: **stop and wait for the user's confirmation before touching the file.** The user may approve all, approve partially, or amend verdicts.
- **Always stop, regardless of tier**: every CHALLENGE (user-only by construction), every out-of-repo write (confirmed individually — named file, exact content), and anything the PRIMARY CHECK flags. A block whose verdict depends on a CHALLENGE answer waits for that answer; independent blocks proceed.

Plan contents per file:
- Before/after estimated line counts.
- A table of blocks: section → verdict → destination (for RELOCATE) → evidence (for DELETE).
- **CHALLENGE items first, grouped by stakes** — not a flat undifferentiated list: (1) rules gating destructive or safety-relevant behavior (git operations, deploy, secrets) first, (2) rules three or more other lines depend on or reference next, (3) everything else batched last as "minor CHALLENGEs" the user can wave through as a block rather than one by one. Within each group, each item as a concrete question: the line, what it claims, what the repo picture shows instead (or which rule it contradicts), and the options (fix to match reality / keep — I'm missing context / delete; a **Correctly-scoped?** item additionally offers: promote to global `~/.claude/CLAUDE.md` / move to `CLAUDE.local.md`; a **Rule-vs-memory?** item additionally offers: belongs in memory — Step 5 defines what each resolution does). The user's answer decides the block's final verdict.
- **Out-of-repo writes are confirmed individually.** Any resolution that writes outside the repo being tidied (e.g. promote-to-global) is presented on its own — named file, exact content shown — and is never bundled into a bulk "approve all"; only in-repo changes may be bulk-approved.
- Anything else you were unsure how to classify, flagged as a question.

## Step 5 — Apply (per the Step 4 gate)

1. **Branching:** immediately before the first commit, explicitly re-check whether the target's own CLAUDE.md defines branch/worktree conventions — do this even for a trivial single-line edit ("it's small" is not an exemption) and even when the file being committed is the CLAUDE.md that defines those conventions (a self-referential edit does not exempt itself from its own rule). If conventions are defined, follow them in full. Only once confirmed the repo defines none does the size-based fallback apply: multi-file changes go on a branch, a single-file compress-only edit may go direct.
2. Execute RELOCATEs first (create/extend destination files, add cross-links both ways, and write the rich-abstract pointer — not a bare "see docs/x.md" link — at the original location) — skip any still blocked by an unresolved Step 0 encryption or CI-dependency flag — then COMPRESS, then DELETE, then update the CLAUDE.md pointers.
3. **Imported-file edit policy** — verdicts can land on imported blocks (Step 2b), but where the edit goes depends on what the imported file is:
   - *In-repo instruction files* (an imported `AGENTS.md`, `docs/git-instructions.md`, or similar file existing to instruct AI tools): edit the imported file directly; never duplicate imported content back into the CLAUDE.md. Step 0's flags extend here — the encryption same-scope verification applies to any imported file edited, and the CI-dependency grep must include imported filenames.
   - *Dual-purpose files* (`@README`, `@package.json`, any imported file whose primary audience/function is not Claude instruction): **never edited by this skill** — a hygiene edit would damage the file's primary function. The finding lands on the **import line** in CLAUDE.md instead (e.g. CHALLENGE: "this imports the full README into every session — intended, or would a slimmer dedicated instruction file serve better?"). Unclear which kind → treat as dual-purpose and CHALLENGE, never guess an edit.
   - *Out-of-repo files* (`@~/...`, external absolute paths): audited, never auto-edited — verdicts surface as individually-confirmed CHALLENGEs per Step 4's out-of-repo rule.
4. **Scope-resolution destinations:** *promote-to-global* appends the individually-confirmed content to `~/.claude/CLAUDE.md`, exactly as shown at Step 4. *Move-to-`CLAUDE.local.md`* creates the file at the repo root if missing and verifies it is gitignored (extend `.gitignore` in the same change if not). *Belongs-in-memory* writes nothing and removes nothing — the line stays untouched and is reported in Step 6; the user performs the move themselves (the documented manual path), and only a **later** run may DELETE the original, once a targeted grep finds the content verifiably present in `MEMORY.md`/topic files (normal DELETE evidence — the skill never removes a line whose destination it didn't write and can't verify).
5. **Keep docs in sync:** grep for every moved/renamed term and update all referencing docs in the same change.
6. **No-loss check:** for each removed line, confirm it is either present at its new home or listed in the plan as a verified DELETE. Re-read the final file top to bottom for coherence.
7. **Iterative self-review loop:** after applying, run a fresh adversarial review of the result — hunt for information loss, broken references, degraded wording, contradictions the edits introduced. Fix findings and re-review until a pass finds nothing. Commit each iteration separately — never batch the loop into one commit.
8. Each iteration ends in its own commit (the first summarizing before/after line counts); merge/publish per the repo's conventions. Update the repo's CHANGELOG if it keeps one.

## Step 6 — Report

Final summary: per file, before → after line counts, blocks relocated (with destinations), blocks deleted (with evidence), any open questions the user deferred, and any lines resolved *belongs-in-memory* — left in place, awaiting the user's manual move.

## Step 7 — Record the run (always, even for analyze-only or report-only runs)

Write **one new file per run** to `${CLAUDE_PLUGIN_DATA}/RUNS-archive/` (create the directory if missing) — never append to a shared file; every run gets its own permanent, standalone log. Name it `<YYYYMMDD>-<HHMM>-<target>.log` in local date/time, where `<target>` is `user` for a run scoped to `~/.claude`, or the target repo's directory basename otherwise (sanitize to `[A-Za-z0-9_-]`, e.g. `job2026`, `Claude-Warp`). On a same-minute, same-target collision, append `-2`, `-3`, ....

```markdown
# claudemd-tidy — run log

- **Date/time:** YYYY-MM-DD HH:MM
- **Target:** <repo name, or "user"> — `<absolute path to the repo root, or ~/.claude>`
- **Processed:** no
- **Target classes:** <which classes this run covered: project / user-level / skills / memory — lets reflection compare instruction performance per class>
- **Memory snapshot:** <path, if a memory edit happened — or "n/a">
- **Commit(s):** <the SHA(s) this run produced — one per apply iteration per the reversibility gate, oldest first — or "n/a — no changes applied" / "n/a — run aborted before any edit">
- **Result:** <before> → <after> lines · <n> relocated · <n> compressed · <n> deleted · <n> challenged (or "analyze-only, not applied")
- **Instructions exercised:** <for each non-KEEP verdict and each CHALLENGE, the SKILL.md step or test that produced it (e.g. "RELOCATE: Step 3 verdict table", "CHALLENGE: Step 2b Consistent?") — or "none (analyze-only found nothing to act on)". This is what lets `/tidyclaudemd:claudemd-tidy-reflect` later tell which instructions are pulling weight across runs and which never fire.>
- **User feedback:** <every amendment, CHALLENGE resolution, and remark the user made, verbatim-ish, each tagged:
  `[general → suggested home: tidy skill / reflect skill / global hygiene rules]` if the lesson would
  hold in other repos, or `[repo-specific]` if it only reflects this repo's context — or "none">
- **Questions asked:** <ambiguities the skill had to ask the user about — or "none">
- **Friction / errors:** <apply-phase problems, blocked commands, missing destinations — or "none">
- **Uncovered cases:** <content that fit no verdict, rules that gave no answer — or "none">
```

Assess generalizability **at recording time, while the context is fresh** — the reflect skill verifies the tag but relies on this first-hand judgment. When in doubt, tag `[general?]` and let reflection decide.

Be honest and specific — these logs are the training data for `/tidyclaudemd:claudemd-tidy-reflect`, which gathers evidence by scanning every `RUNS-archive/*.log` file for `**Processed:** no`. If any field is non-empty besides Result, suggest the user run `/tidyclaudemd:claudemd-tidy-reflect` to fold the lesson back into this skill.
