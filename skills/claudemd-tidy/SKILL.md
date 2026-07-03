---
name: claudemd-tidy
description: Scan every CLAUDE.md in the current repo against the global "CLAUDE.md hygiene" rules and slim it by relocating/compressing content — never losing information. Use when the user asks to tidy, slim, audit, or clean up a CLAUDE.md.
version: 0.9.1
---

# /tidyclaudemd:claudemd-tidy

Audit and slim the CLAUDE.md file(s) of the current repo. Two phases: **analyze & propose** (always), then **apply** (only after the user confirms).

## Argument

Optional path to a specific CLAUDE.md, and/or `--report` to run report mode (below) instead of a full tidy. If no path is given, process every CLAUDE.md in the current repo (root + nested).

## Report mode (`--report`)

A cheap, no-plan, no-confirmation, no-edits pre-check for "how bad is this CLAUDE.md right now?" — skips Step 2's survey and Step 2b's five-test interrogation entirely. Runs Step 0 (Preflight) as normal, then reports two clearly-separated tiers of output and stops:

- **Mechanical checks** (scriptable, no judgment): line/token count per file vs. the global hygiene guardrail (~150 lines); non-English/CJK content flagged with an estimated token-overhead cost (non-English content runs roughly 30-50% more tokens per instruction than equivalent English); a session-cost estimate (per-request token cost × an assumed 30-turn session).
- **A rough verdict-mix impression** from a shallow read only — an approximate KEEP/COMPRESS/RELOCATE/DELETE/CHALLENGE tally. Label this explicitly as unverified judgment in the output, never with the same confidence as the mechanical counts above — verdict classification without Step 2b's real interrogation is a guess, not an analysis.

A report-only run still appends a Step 7 run record (tag it analyze-only in the Result field) and explicitly states in its output that it is a heuristic pre-check, not a substitute for a full tidy.

## Step 0 — Preflight

1. Confirm you are inside a git repo; if not, ask which file to tidy and skip git steps.
2. Sync per the global working defaults: `git fetch`, check branch/ahead-behind/dirty state.
3. Check repo visibility (`gh repo view --json visibility` or inspect the remote). If public or unknown → the **PRIMARY CHECK** in `~/.claude/CLAUDE.md` applies to every file this skill writes, including relocated content.
4. **Encryption check.** Scan for git-crypt/SOPS filters in `.gitattributes` or an `.age` key reference. If found, flag it: in Step 3, any encryption-unlock instructions in the CLAUDE.md are force-classified **KEEP** (never RELOCATE — moving them risks a chicken-and-egg lock-out); any other RELOCATE destination must be verified as covered by the same encryption scope as the source before Step 5 executes it.
5. **CI-dependency check.** Scan CI config (`.github/workflows/` and any other CI directories at the repo root) for scripts that reference `CLAUDE.md` by filename (e.g. a script that greps its content for a required phrase). If found, flag the content those scripts appear to depend on: it's ineligible for RELOCATE in Step 5 without the user's explicit confirmation in Step 4 that the CI dependency is accounted for.

## Step 1 — Load the rules

Read the **"CLAUDE.md hygiene — keep every CLAUDE.md slim"** section of `~/.claude/CLAUDE.md`. Those numbered rules are the single source of truth for this skill — do not work from memory of them.

**If the section is missing** (first-time setup): append the contents of `${CLAUDE_PLUGIN_ROOT}/HYGIENE-RULES-TEMPLATE.md` verbatim to the end of `~/.claude/CLAUDE.md` as a new section — do not overwrite or reorder anything already in that file. Tell the user you did this and why. This is a **one-time bootstrap**: from this point on, `~/.claude/CLAUDE.md` is the sole source of truth, exactly as if the section had always been there — this skill never reads the bundled template again after the first install, and the reflect skill only ever proposes edits to the global file, never to the template.

Also read the **rest** of `~/.claude/CLAUDE.md` in full (every earlier-loaded scope, not just the hygiene section) — needed for Step 2b's **Redundant-by-order?** question, which can't be answered from the hygiene section alone.

## Step 2 — Build a complete picture of the repo

**Do not analyze a single CLAUDE.md line before this step is done.** Every verdict in Step 3 must be grounded in what the repo *actually* is, not in what the CLAUDE.md claims it is.

1. Find targets: `git ls-files '*CLAUDE.md' 'CLAUDE.md'` plus an untracked-file check; include nested ones. **Also** glob the filesystem directly for common gitignored personal-override filenames (starting with `CLAUDE.local.md`) — `git ls-files` and a standard untracked-file check both skip gitignored paths, so a personal-override convention would otherwise never surface. List any found explicitly in the Step 4 plan even if the user then chooses to exclude them — that must be a stated choice, not a silent gap. For each target found, record line count, word count, section list.
2. **Fixed general-orientation pass** (always, regardless of CLAUDE.md size): top-level `README.md`; a listing — not a deep read — of `.claude/commands/` and `.claude/skills/`; `.claude/settings.json`.
3. **Claim-driven verification**: extract every concrete claim each CLAUDE.md makes — file paths, command names, workflow descriptions, tool/skill references — then verify only those against the repo (targeted file/path lookups, package manifests or CI config only if a claim points there, git history only if a claim makes an assertion about change frequency or ownership). Do not read anything the CLAUDE.md makes no claim about; this is the survey's cost control — it scales with the CLAUDE.md's own length, not the repo's size.
4. Inventory the possible relocation homes: `.claude/commands/` and `.claude/skills/` (per-task procedures), `docs/` or `plans/` (background/design), `README.md` (human-facing description).
5. **Comprehension gate** — before moving on, you must be able to answer: what does this repo do; how is it built/run/deployed; which agents, hooks, and scheduled processes operate on it; which conventions demonstrably hold in the tree (naming, layout, workflows) and which appear only in the CLAUDE.md. If you can't, **keep resolving unverified claims** from bullet 3 — never expand into an open-ended survey.

## Step 2b — Critically interrogate every line

Walk each CLAUDE.md **line by line** and question whether it makes sense, before any slimming verdict. For each rule/claim ask:

- **True?** Does it match the repo picture — do the files, commands, paths, and behaviors it references exist and work as described?
- **Live?** Is it still needed, or does it govern a workflow/file that no longer exists or was superseded?
- **Consistent?** Does it contradict another rule in this file, a nested CLAUDE.md, a skill, or the repo's observable conventions? Actively probe for common contradiction **shapes**, not just whatever a line-by-line read happens to surface: autonomy vs. ask-first, a stated default vs. an exception elsewhere, two rules naming different values for the same threshold/setting, "always X" vs. a documented case where the repo does not-X.
- **Actionable?** Is it unambiguous enough that two sessions would follow it the same way?
- **Redundant-by-order?** Does a global-scope file (or any other earlier-loaded rule, per Claude Code's config load order: global CLAUDE.md → global rules → project CLAUDE.md → project rules → Memory) already establish this same intent, making this line pure repetition regardless of exact wording?

A line that fails True / Live / Consistent / Actionable is **suspect** — never silently fix, keep, or drop it; route it to the CHALLENGE verdict below. A line that fails only **Redundant-by-order** is not suspect in the same sense (it isn't wrong, just repeated) — it routes to COMPRESS or CHALLENGE per the Step 3 verdict table, never straight to DELETE.

## Step 3 — Analyze

Walk each CLAUDE.md section by section and classify every block with exactly one verdict, using the Step 2 repo picture and the Step 2b interrogation results:

| Verdict | Meaning | Test |
|---|---|---|
| **KEEP** | Behavior-changing rule, already terse | Passes the behavior test and all Step 2b questions as-is |
| **COMPRESS** | Rule stays here, rewritten tighter | Same meaning expressible in fewer lines (prose → bullets/table, redundant examples dropped), **or** fails only **Redundant-by-order** and trimming to a one-line acknowledgment of the earlier-loaded rule preserves genuine project-specific nuance |
| **RELOCATE** | Detail moves to its proper home; a **rich-abstract pointer** stays — a short synopsis stating the concrete fact(s) a reader would otherwise need to open the sub-doc for (the specific convention, threshold, or command, not just that a sub-doc exists), plus the link | Procedure/step detail whose natural owner is a skill, `docs/`, or README |
| **DELETE** | Removed outright | Duplicated verbatim elsewhere (cite location), **or** dead and grep-*confirmed* gone (cite the failed lookup — a rename counts as confirmed if the new location is found and cited instead) |
| **CHALLENGE** | Suspect — raised to the user as a question, never resolved unilaterally | Failed a Step 2b question but **not** grep-confirmable either way: ambiguous liveness, possible-but-unconfirmed rename, contradiction between rules, or wording too vague to act on; **or** fails only Redundant-by-order with no confirmed project-specific nuance (cite the earlier-loaded rule; the user decides compress-further vs. delete — never route this straight to DELETE, which requires a verbatim duplicate or grep-confirmed dead reference, neither of which a same-intent-different-wording match is) — **or** an otherwise-RELOCATE-eligible block blocked by an unresolved Step 0 encryption/CI-dependency flag (see Rules of evidence below) |

DELETE and CHALLENGE are mutually exclusive by evidence strength, not by which Step 2b question failed: a **confirmed** dead reference is DELETE; anything the grep can't settle is CHALLENGE. If a search is inconclusive, default to CHALLENGE — never guess a DELETE.

Rules of evidence:
- Before any **DELETE**, verify: grep the repo for the referenced file/command/term. Cite the evidence (the duplicate's location, or the failed lookup proving it's gone) in the plan.
- Before any **RELOCATE**, identify the exact destination file and whether it already covers the content (then it's a DELETE-as-duplicate with a pointer check, not a move).
- Content whose only sin is verbosity is COMPRESS, never DELETE.
- A block flagged by Step 0's encryption or CI-dependency check cannot be RELOCATEd until that flag's condition is satisfied (matching encryption scope confirmed, or the user explicitly confirms the CI dependency is handled in Step 4) — until then it routes to CHALLENGE, not RELOCATE.

## Step 4 — Report and confirm (stop point)

Present a plan per file:
- Before/after estimated line counts.
- A table of blocks: section → verdict → destination (for RELOCATE) → evidence (for DELETE).
- **CHALLENGE items first, grouped by stakes** — not a flat undifferentiated list: (1) rules gating destructive or safety-relevant behavior (git operations, deploy, secrets) first, (2) rules three or more other lines depend on or reference next, (3) everything else batched last as "minor CHALLENGEs" the user can wave through as a block rather than one by one. Within each group, each item as a concrete question: the line, what it claims, what the repo picture shows instead (or which rule it contradicts), and the options (fix to match reality / keep — I'm missing context / delete). The user's answer decides the block's final verdict.
- Anything else you were unsure how to classify, flagged as a question.

**Stop and wait for the user's confirmation before touching any file.** The user may approve all, approve partially, or amend verdicts.

## Step 5 — Apply (only after confirmation)

1. **Branching:** follow the repo's own conventions if its CLAUDE.md defines any (branch/worktree rules); otherwise, multi-file changes go on a branch, a single-file compress-only edit may go direct.
2. Execute RELOCATEs first (create/extend destination files, add cross-links both ways, and write the rich-abstract pointer — not a bare "see docs/x.md" link — at the original location) — skip any still blocked by an unresolved Step 0 encryption or CI-dependency flag — then COMPRESS, then DELETE, then update the CLAUDE.md pointers.
3. **Keep docs in sync:** grep for every moved/renamed term and update all referencing docs in the same change.
4. **No-loss check:** for each removed line, confirm it is either present at its new home or listed in the plan as a verified DELETE. Re-read the final CLAUDE.md top to bottom for coherence.
5. Commit with a message summarizing before/after line counts; merge/publish per the repo's conventions. Update the repo's CHANGELOG if it keeps one.

## Step 6 — Report

Final summary: per file, before → after line counts, blocks relocated (with destinations), blocks deleted (with evidence), and any open questions the user deferred.

## Step 7 — Record the run (always, even for analyze-only or report-only runs)

Append a run record at the **top** of `${CLAUDE_PLUGIN_DATA}/RUNS.md` (create the file with a `# claudemd-tidy — run records` header if missing):

```markdown
## YYYY-MM-DD — <repo> (<files processed>)
- **Processed:** no
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

Be honest and specific — these records are the training data for `/tidyclaudemd:claudemd-tidy-reflect`. If any field is non-empty besides Result, suggest the user run `/tidyclaudemd:claudemd-tidy-reflect` to fold the lesson back into this skill.
