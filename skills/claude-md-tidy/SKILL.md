---
name: claude-md-tidy
description: Scan every CLAUDE.md in the current repo against the global "CLAUDE.md hygiene" rules and slim it by relocating/compressing content — never losing information. Use when the user asks to tidy, slim, audit, or clean up a CLAUDE.md.
version: 0.3.1
---

# /claude-md-tidy

Audit and slim the CLAUDE.md file(s) of the current repo. Two phases: **analyze & propose** (always), then **apply** (only after the user confirms).

## Argument

Optional path to a specific CLAUDE.md. If omitted, process every CLAUDE.md in the current repo (root + nested).

## Step 0 — Preflight

1. Confirm you are inside a git repo; if not, ask which file to tidy and skip git steps.
2. Sync per the global working defaults: `git fetch`, check branch/ahead-behind/dirty state.
3. Check repo visibility (`gh repo view --json visibility` or inspect the remote). If public or unknown → the **PRIMARY CHECK** in `~/.claude/CLAUDE.md` applies to every file this skill writes, including relocated content.

## Step 1 — Load the rules

Read the **"CLAUDE.md hygiene — keep every CLAUDE.md slim"** section of `~/.claude/CLAUDE.md`. Those numbered rules are the single source of truth for this skill — do not work from memory of them. If the section is missing, stop and report.

## Step 2 — Build a complete picture of the repo

**Do not analyze a single CLAUDE.md line before this step is done.** Every verdict in Step 3 must be grounded in what the repo *actually* is, not in what the CLAUDE.md claims it is.

1. Find targets: `git ls-files '*CLAUDE.md' 'CLAUDE.md'` plus an untracked-file check; include nested ones. For each, record line count, word count, section list.
2. Survey the repo: full file tree; `README.md` and everything under `docs/`; every skill/command in `.claude/commands/` and `.claude/skills/`; `.claude/settings.json` (hooks, permissions); package manifests and build/CI config; recent git history (what actually changes, and who — humans, scheduled agents, CI).
3. Inventory the possible relocation homes: `.claude/commands/` and `.claude/skills/` (per-task procedures), `docs/` or `plans/` (background/design), `README.md` (human-facing description).
4. **Comprehension gate** — before moving on, you must be able to answer: what does this repo do; how is it built/run/deployed; which agents, hooks, and scheduled processes operate on it; which conventions demonstrably hold in the tree (naming, layout, workflows) and which appear only in the CLAUDE.md. If you can't, keep reading.

## Step 2b — Critically interrogate every line

Walk each CLAUDE.md **line by line** and question whether it makes sense, before any slimming verdict. For each rule/claim ask:

- **True?** Does it match the repo picture — do the files, commands, paths, and behaviors it references exist and work as described?
- **Live?** Is it still needed, or does it govern a workflow/file that no longer exists or was superseded?
- **Consistent?** Does it contradict another rule in this file, a nested CLAUDE.md, a skill, or the repo's observable conventions?
- **Actionable?** Is it unambiguous enough that two sessions would follow it the same way?

A line that fails any of these is **suspect** — never silently fix, keep, or drop it; route it to the CHALLENGE verdict below.

## Step 3 — Analyze

Walk each CLAUDE.md section by section and classify every block with exactly one verdict, using the Step 2 repo picture and the Step 2b interrogation results:

| Verdict | Meaning | Test |
|---|---|---|
| **KEEP** | Behavior-changing rule, already terse | Passes the behavior test and all Step 2b questions as-is |
| **COMPRESS** | Rule stays here, rewritten tighter | Same meaning expressible in fewer lines (prose → bullets/table, redundant examples dropped) |
| **RELOCATE** | Detail moves to its proper home; a one-line pointer stays | Procedure/step detail whose natural owner is a skill, `docs/`, or README |
| **DELETE** | Removed outright | Duplicated verbatim elsewhere (cite location), **or** dead and grep-*confirmed* gone (cite the failed lookup — a rename counts as confirmed if the new location is found and cited instead) |
| **CHALLENGE** | Suspect — raised to the user as a question, never resolved unilaterally | Failed a Step 2b question but **not** grep-confirmable either way: ambiguous liveness, possible-but-unconfirmed rename, contradiction between rules, or wording too vague to act on |

DELETE and CHALLENGE are mutually exclusive by evidence strength, not by which Step 2b question failed: a **confirmed** dead reference is DELETE; anything the grep can't settle is CHALLENGE. If a search is inconclusive, default to CHALLENGE — never guess a DELETE.

Rules of evidence:
- Before any **DELETE**, verify: grep the repo for the referenced file/command/term. Cite the evidence (the duplicate's location, or the failed lookup proving it's gone) in the plan.
- Before any **RELOCATE**, identify the exact destination file and whether it already covers the content (then it's a DELETE-as-duplicate with a pointer check, not a move).
- Content whose only sin is verbosity is COMPRESS, never DELETE.

## Step 4 — Report and confirm (stop point)

Present a plan per file:
- Before/after estimated line counts.
- A table of blocks: section → verdict → destination (for RELOCATE) → evidence (for DELETE).
- **CHALLENGE items first**, each as a concrete question: the line, what it claims, what the repo picture shows instead (or which rule it contradicts), and the options (fix to match reality / keep — I'm missing context / delete). The user's answer decides the block's final verdict.
- Anything else you were unsure how to classify, flagged as a question.

**Stop and wait for the user's confirmation before touching any file.** The user may approve all, approve partially, or amend verdicts.

## Step 5 — Apply (only after confirmation)

1. **Branching:** follow the repo's own conventions if its CLAUDE.md defines any (branch/worktree rules); otherwise, multi-file changes go on a branch, a single-file compress-only edit may go direct.
2. Execute RELOCATEs first (create/extend destination files, add cross-links both ways), then COMPRESS, then DELETE, then update the CLAUDE.md pointers.
3. **Keep docs in sync:** grep for every moved/renamed term and update all referencing docs in the same change.
4. **No-loss check:** for each removed line, confirm it is either present at its new home or listed in the plan as a verified DELETE. Re-read the final CLAUDE.md top to bottom for coherence.
5. Commit with a message summarizing before/after line counts; merge/publish per the repo's conventions. Update the repo's CHANGELOG if it keeps one.

## Step 6 — Report

Final summary: per file, before → after line counts, blocks relocated (with destinations), blocks deleted (with evidence), and any open questions the user deferred.

## Step 7 — Record the run (always, even for analyze-only runs)

Append a run record at the **top** of `~/.claude/skills/claude-md-tidy/RUNS.md` (create the file with a `# claude-md-tidy — run records` header if missing):

```markdown
## YYYY-MM-DD — <repo> (<files processed>)
- **Processed:** no
- **Result:** <before> → <after> lines · <n> relocated · <n> compressed · <n> deleted · <n> challenged (or "analyze-only, not applied")
- **User feedback:** <every amendment, CHALLENGE resolution, and remark the user made, verbatim-ish, each tagged:
  `[general → suggested home: tidy skill / reflect skill / global hygiene rules]` if the lesson would
  hold in other repos, or `[repo-specific]` if it only reflects this repo's context — or "none">
- **Questions asked:** <ambiguities the skill had to ask the user about — or "none">
- **Friction / errors:** <apply-phase problems, blocked commands, missing destinations — or "none">
- **Uncovered cases:** <content that fit no verdict, rules that gave no answer — or "none">
```

Assess generalizability **at recording time, while the context is fresh** — the reflect skill verifies the tag but relies on this first-hand judgment. When in doubt, tag `[general?]` and let reflection decide.

Be honest and specific — these records are the training data for `/claude-md-tidy-reflect`. If any field is non-empty besides Result, suggest the user run `/claude-md-tidy-reflect` to fold the lesson back into this skill.
