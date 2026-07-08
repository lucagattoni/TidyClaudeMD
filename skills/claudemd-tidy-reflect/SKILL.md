---
name: claudemd-tidy-reflect
description: Self-improvement loop for TidyClaudeMD. Learns from recorded tidy runs and concrete CLAUDE.md instances, then applies evidence-backed improvements to the tidy skill itself — bumping the version and CHANGELOG. Use after a tidy run surfaced friction, or when asked to improve/reflect on claudemd-tidy.
version: 0.14.0
---

# /tidyclaudemd:claudemd-tidy-reflect

Learn from concrete tidy experience and fold the lessons back into `${CLAUDE_PLUGIN_ROOT}/skills/claudemd-tidy/SKILL.md`. **Every change must be traceable to evidence from a real run or instance — no evidence, no change.**

## Argument

Optional: a path to a specific CLAUDE.md to study as a fresh instance (analysis only, no edits to it), or a run date from `RUNS.md` to (re)process. Default: all records marked `**Processed:** no` in `${CLAUDE_PLUGIN_DATA}/RUNS.md`, plus the current session's tidy run if one just happened.

## Step 0 — Load the suite

Read all of: `${CLAUDE_PLUGIN_ROOT}/skills/claudemd-tidy/SKILL.md`, this skill's own `SKILL.md`, `${CLAUDE_PLUGIN_ROOT}/README.md`, `${CLAUDE_PLUGIN_ROOT}/CHANGELOG.md`, `${CLAUDE_PLUGIN_DATA}/RUNS.md`, and the "CLAUDE.md hygiene" section of `~/.claude/CLAUDE.md`. Note the current suite version.

## Step 1 — Gather evidence

- Unprocessed run records from `RUNS.md`.
- If invoked right after a tidy run in this session: the live experience (user amendments, questions, friction) — even before it's recorded.
- If given a CLAUDE.md path: dry-run the tidy skill's analyze phase against it and note every point where the current instructions are ambiguous, silent, or force a question.
- **An ad-hoc lesson described directly in the invoking conversation** — not tied to any recorded run or dry-run, e.g. feedback about CLAUDE.md quality or suite design raised in an ordinary conversation. The conversation itself (and, where one exists, a doc such as a plan/backlog file) stands in as the citation in place of a `RUNS.md` record.
- Whether new evidence **corroborates or contradicts** an existing `provisional` lesson (records marked `Processed: yes (vX.Y.Z, provisional)` — see Step 5). A contradiction is itself top-priority evidence to revisit or demote that lesson, cited the same way a fresh lesson would be; a second, independent corroboration promotes it.

**If there is no evidence, report that and stop.** Never invent improvements from first principles.

## Step 2 — Extract lessons

Mine the evidence for signals, strongest first:

1. **User feedback tagged `[general …]` or `[general?]` in the run record** → the tidy run already judged it generalizable while the context was fresh; verify the tag (don't trust it blindly), then treat it as the highest-priority lesson. `[repo-specific]` items are only suggested to that repo, never folded into the suite.
2. **User overrode a verdict** → the classification rules mis-sorted a real block; what distinguishing feature did they see that the rules miss?
3. **The skill had to ask a question** the instructions should have answered → a gap to close.
4. **Apply-phase friction or error** (blocked command, missing destination, sync conflict) → a preflight or apply step to harden.
5. **Content that fit no verdict or no destination** → the taxonomy needs extending.
6. **An existing instruction in `claudemd-tidy/SKILL.md` never triggered across the processed run records** — never cited as the reason a CHALLENGE/DELETE/RELOCATE fired, never referenced in feedback — is a **pruning candidate**. Requires the same evidence discipline as an addition: cite which records were checked and found the instruction inert before proposing its removal in Step 5.

Each candidate lesson must pass all three tests, or be discarded (and reported as discarded, with the reason):
- **Generalizable** — would improve tidy runs in *other* repos, not just this one.
- **Traceable** — cites the specific run record or instance that motivated it.
- **Correctly homed** — see Step 3.

## Step 3 — Route each lesson to its home

- **Tidy mechanics** (scanning, verdict tests, apply order, recording) → `claudemd-tidy/SKILL.md`.
- **Reflection mechanics** (evidence gathering, lesson tests, this workflow) → this skill's own `SKILL.md` — the loop applies to itself.
- **What "good CLAUDE.md content" means** (the hygiene rules themselves) → the "CLAUDE.md hygiene" section of `~/.claude/CLAUDE.md`. Never fork a private copy of those rules into the skill; propose the global edit and get explicit user confirmation, since that rulebook governs every session.
- **Repo-specific quirks** → that repo's own CLAUDE.md (suggest it to the user); never encode them into the general skill.

## Step 4 — Invariant gate

These invariants (documented in `README.md` → Invariants) may **never be weakened or removed autonomously**: the Step-4 stop-and-confirm, the no-loss guarantee, evidence-before-DELETE, the public-repo PRIMARY CHECK, hygiene rules single-sourced at runtime in the global CLAUDE.md (the plugin's one-time bootstrap template is not an exception — see README invariant 5), and no-evidence-no-change (this skill's own). A lesson that would touch one of them is presented to the user with pros/cons and applied only on their explicit approval. Everything else may be applied autonomously.

## Step 5 — Apply

1. Edit the routed file(s) with **minimal diffs** — surgical insertions/rewrites, each mapped to one lesson. A **pruning** lesson (Step 2, signal 6) removes the inert instruction from `claudemd-tidy/SKILL.md` the same way an addition inserts one — cite the checked records in the CHANGELOG entry (sub-step 3).
2. **Bump the suite version** (one semver for the whole suite, kept identical in both skills' frontmatter):
   - **PATCH** — clarified wording, tightened an existing test, fixed a record format.
   - **MINOR** — new step, new verdict, new signal, new capability.
   - **MAJOR** — changed workflow contract (phases, stop points, file layout). Always requires user approval, independent of the invariant gate.
3. Add a `CHANGELOG.md` entry under the new version: each change on one line, ending with its evidence citation `(run: YYYY-MM-DD <repo>)`. A lesson drawn from a **single** run record is marked **provisional**: `(provisional, 1 occurrence — run: YYYY-MM-DD <repo>)`. When a second, independent run corroborates a provisional lesson, promote it: rewrite the original line to drop the `provisional` tag and cite both runs, rather than adding a second line for the same lesson.
4. **Check the README bullet for every SKILL.md step touched this pass** — not just "if a documented feature changed" (easy to judge false when the change is subtle). If the edited step has a corresponding README bullet, update it in the same diff: paraphrase behavior and intent, never restate a step's normative test or wording verbatim. Never leave `README.md` describing superseded behavior.
5. Mark consumed records in `RUNS.md`: `**Processed:** yes (vX.Y.Z)` — or `**Processed:** yes (vX.Y.Z, provisional)` if any lesson drawn from that record was applied provisionally in sub-step 3. When a provisional lesson is later promoted, update the *originating* record's marker in place to drop `, provisional`.
6. **Archive fully-processed records.** A record is an archive candidate once it is `Processed: yes` **and not** `provisional` (a still-pending-corroboration provisional record is never archived). A user may also add `**Pinned:** yes` to any record to exempt it from archiving indefinitely, regardless of processed/provisional state. Keep the 3 most recent archive-eligible records in `RUNS.md`; move the rest, oldest first, into a dated file inside `${CLAUDE_PLUGIN_DATA}/RUNS-archive/` (create the directory if missing) — never delete. `RUNS.md` stays bounded to recent/active records plus anything still provisional or pinned; archived records remain fully readable, just no longer in the always-loaded file.
7. **Keep the plugin manifest in sync.** Update `.claude-plugin/plugin.json`'s `version` field to match the bump in sub-step 2, in the same pass. (`.claude-plugin/marketplace.json` has no per-version field for a same-repo plugin — install always resolves to whatever `plugin.json` currently says — so there's nothing there to update.)

## Step 6 — Report

State: old → new version, each change made with its evidence, lessons discarded (with which test they failed), and any invariant-touching or global-rulebook proposals awaiting the user's decision.
