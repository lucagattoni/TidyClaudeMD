---
name: claude-md-tidy-reflect
description: Self-improvement loop for the claude-md-tidy suite. Learns from recorded tidy runs and concrete CLAUDE.md instances, then applies evidence-backed improvements to the tidy skill itself — bumping the version and CHANGELOG. Use after a tidy run surfaced friction, or when asked to improve/reflect on claude-md-tidy.
version: 0.3.1
---

# /claude-md-tidy-reflect

Learn from concrete tidy experience and fold the lessons back into `~/.claude/skills/claude-md-tidy/SKILL.md`. **Every change must be traceable to evidence from a real run or instance — no evidence, no change.**

## Argument

Optional: a path to a specific CLAUDE.md to study as a fresh instance (analysis only, no edits to it), or a run date from `RUNS.md` to (re)process. Default: all records marked `**Processed:** no` in `~/.claude/skills/claude-md-tidy/RUNS.md`, plus the current session's tidy run if one just happened.

## Step 0 — Load the suite

Read all of: `claude-md-tidy/SKILL.md`, this skill's own `SKILL.md`, `claude-md-tidy/README.md`, `claude-md-tidy/CHANGELOG.md`, `claude-md-tidy/RUNS.md`, and the "CLAUDE.md hygiene" section of `~/.claude/CLAUDE.md`. Note the current suite version.

## Step 1 — Gather evidence

- Unprocessed run records from `RUNS.md`.
- If invoked right after a tidy run in this session: the live experience (user amendments, questions, friction) — even before it's recorded.
- If given a CLAUDE.md path: dry-run the tidy skill's analyze phase against it and note every point where the current instructions are ambiguous, silent, or force a question.

**If there is no evidence, report that and stop.** Never invent improvements from first principles.

## Step 2 — Extract lessons

Mine the evidence for signals, strongest first:

1. **User feedback tagged `[general …]` or `[general?]` in the run record** → the tidy run already judged it generalizable while the context was fresh; verify the tag (don't trust it blindly), then treat it as the highest-priority lesson. `[repo-specific]` items are only suggested to that repo, never folded into the suite.
2. **User overrode a verdict** → the classification rules mis-sorted a real block; what distinguishing feature did they see that the rules miss?
3. **The skill had to ask a question** the instructions should have answered → a gap to close.
4. **Apply-phase friction or error** (blocked command, missing destination, sync conflict) → a preflight or apply step to harden.
5. **Content that fit no verdict or no destination** → the taxonomy needs extending.

Each candidate lesson must pass all three tests, or be discarded (and reported as discarded, with the reason):
- **Generalizable** — would improve tidy runs in *other* repos, not just this one.
- **Traceable** — cites the specific run record or instance that motivated it.
- **Correctly homed** — see Step 3.

## Step 3 — Route each lesson to its home

- **Tidy mechanics** (scanning, verdict tests, apply order, recording) → `claude-md-tidy/SKILL.md`.
- **Reflection mechanics** (evidence gathering, lesson tests, this workflow) → this skill's own `SKILL.md` — the loop applies to itself.
- **What "good CLAUDE.md content" means** (the hygiene rules themselves) → the "CLAUDE.md hygiene" section of `~/.claude/CLAUDE.md`. Never fork a private copy of those rules into the skill; propose the global edit and get explicit user confirmation, since that rulebook governs every session.
- **Repo-specific quirks** → that repo's own CLAUDE.md (suggest it to the user); never encode them into the general skill.

## Step 4 — Invariant gate

These invariants (documented in `claude-md-tidy/README.md` → Invariants) may **never be weakened or removed autonomously**: the Step-4 stop-and-confirm, the no-loss guarantee, evidence-before-DELETE, the public-repo PRIMARY CHECK, hygiene rules single-sourced in the global CLAUDE.md, and no-evidence-no-change (this skill's own). A lesson that would touch one of them is presented to the user with pros/cons and applied only on their explicit approval. Everything else may be applied autonomously.

## Step 5 — Apply

1. Edit the routed file(s) with **minimal diffs** — surgical insertions/rewrites, each mapped to one lesson.
2. **Bump the suite version** (one semver for the whole suite, kept identical in both skills' frontmatter):
   - **PATCH** — clarified wording, tightened an existing test, fixed a record format.
   - **MINOR** — new step, new verdict, new signal, new capability.
   - **MAJOR** — changed workflow contract (phases, stop points, file layout). Always requires user approval, independent of the invariant gate.
3. Add a `CHANGELOG.md` entry under the new version: each change on one line, ending with its evidence citation `(run: YYYY-MM-DD <repo>)`.
4. Update `README.md` in the same pass if any documented feature changed — never leave it describing superseded behavior.
5. Mark consumed records in `RUNS.md`: `**Processed:** yes (vX.Y.Z)`.

## Step 6 — Report

State: old → new version, each change made with its evidence, lessons discarded (with which test they failed), and any invariant-touching or global-rulebook proposals awaiting the user's decision.
