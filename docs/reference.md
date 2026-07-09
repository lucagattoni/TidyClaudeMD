# TidyClaudeMD — reference manual

Everything the two skills do, in detail. New to Claude Code file types? Start with [claude-code-concepts.md](claude-code-concepts.md). For the short pitch and install steps, see the [README](../README.md).

- [How a tidy run works](#how-a-tidy-run-works)
- [Target classes](#target-classes)
- [The seven interrogation tests](#the-seven-interrogation-tests)
- [The five verdicts](#the-five-verdicts)
- [Preflight checks](#preflight-checks)
- [Applying changes](#applying-changes)
- [Run logs](#run-logs)
- [The reflect skill](#the-reflect-skill)
- [Invariants](#invariants)
- [Optional companion hook](#optional-companion-hook)

## How a tidy run works

Every run — whatever target class it covers — goes through the same eight phases:

| # | Phase | What happens |
|---|---|---|
| 1 | Preflight | Checks the session isn't running a stale plugin version, syncs git, checks repo visibility and encryption/CI dependencies. See [Preflight checks](#preflight-checks). |
| 2 | Load the rules | Reads the hygiene rules from your global `~/.claude/CLAUDE.md` — the skill has no rules baked into itself. |
| 3 | Build a repo picture | A cheap orientation pass plus targeted verification of only the concrete claims the target file makes — never an open-ended read of the whole repo. |
| 4 | Interrogate every line | Each line is checked against seven tests. See [The seven interrogation tests](#the-seven-interrogation-tests). |
| 5 | Assign a verdict | Each block gets exactly one of five verdicts. See [The five verdicts](#the-five-verdicts). |
| 6 | Report and gate | The full plan is always shown. What happens next depends on the reversibility gate — see [Applying changes](#applying-changes). |
| 7 | Apply | Executes RELOCATE → COMPRESS → DELETE, in that order, then a no-loss check and a self-review loop. |
| 8 | Record | Writes a permanent log of the run. See [Run logs](#run-logs). |

Report mode (`--report`) is a cheap shortcut that runs phase 1 only, then a shallow mechanical pre-check (line/token counts, an unverified verdict-mix guess) — never a substitute for a real run, but useful to gauge "how bad is this file right now?" before committing to a full pass.

## Target classes

TidyClaudeMD doesn't only tidy project `CLAUDE.md` files. Pass a flag to target a different class:

| Class | Flag | What it covers | Notable behavior |
|---|---|---|---|
| Project CLAUDE.md | *(default)* | `./CLAUDE.md`, `./.claude/CLAUDE.md`, nested files, `CLAUDE.local.md` | Sweeps every CLAUDE.md in the repo, including gitignored personal-override files a plain `git ls-files` would miss. |
| User-level | `--user` | Your global `~/.claude/CLAUDE.md` and `~/.claude/rules/` | The "is this the right scope?" question inverts: is this line genuinely universal, or does it belong in one project instead? |
| Skills | `--skills` | Project or user-level `SKILL.md` files | Never touches this suite's own two skills — those only change through the reflect loop. Adds three skill-specific tests (below). |
| Memory | `--memory` | The current project's `MEMORY.md` and topic files | Always snapshots the whole memory directory first — no snapshot, no edit. Memory content never lands in a repo-tracked file. |

`--all` runs every class in scope on the current machine. `--report` composes with any class flag.

**Skills-class tests**, on top of the seven below: does the description say what the skill does *and* when to trigger it (it's the only always-loaded part)? Does bulk reference/example content belong in a lazily-read supporting file instead of inline (progressive disclosure)? Is the frontmatter valid — YAML sane, `name:` matching the directory, referenced files actually present?

Beyond those per-file tests, a **sweep-level pass** runs once across every skill found in the run: are two skills byte-identical (Duplicate?), the same body with different frontmatter (Near-duplicate?), the same `name:` colliding across skills or scopes in a way Claude Code doesn't already resolve on its own (Name-conflict?), or plausibly triggered by the same request (Trigger-overlap?, mechanically pre-filtered, then judged directly). All four route to CHALLENGE — never an automatic deletion of a whole skill — with this suite's own two skills participating only as comparison references, never as an editable copy.

**Memory-class tests**: is `MEMORY.md` staying a genuine index (one line per memory, detail pushed to topic files) rather than growing into a second CLAUDE.md? Is the same fact duplicated across topic files? Are stale, no-longer-true discovered facts pruned? Does an entry read like a deliberate every-session rule that should be *promoted* to CLAUDE.md instead (the mirror image of the Rule-vs-memory? test below)?

**`@import` handling.** The audit follows the *effective* content of a CLAUDE.md, not just its literal text: live `@path` imports (an `@AGENTS.md`, a `@docs/git-instructions.md`) are detected and their content interrogated like inline text, up to the four-hop limit Claude Code itself documents. A code-span mention like `` `@README` `` is never treated as an import. Imports pointing outside the repo are flagged as possibly inert, since Claude Code can silently disable them if you once declined its approval dialog. Editing an imported file follows a strict policy: an in-repo instruction file (`AGENTS.md`) is edited directly; a dual-purpose file (`README`, `package.json`) is never edited — the finding lands on the import line instead; an external file is only ever raised as an individually-confirmed question, never auto-edited.

## The seven interrogation tests

Before any line gets a verdict, it's checked against all seven:

| Test | Question |
|---|---|
| True? | Do the files, commands, and paths it references actually exist and work as described? |
| Live? | Is it still needed, or does it govern something superseded or gone? |
| Consistent? | Does it contradict another rule — in this file, a nested CLAUDE.md, an unconditional `.claude/rules/` file, a skill, or the repo's own observable conventions? |
| Actionable? | Is it unambiguous enough that two different sessions would follow it identically? |
| Redundant-by-order? | Does an earlier-loaded scope (global CLAUDE.md, global rules, then this file) already establish the same intent, in any wording? |
| Correctly-scoped? | Is this genuinely about *this* repo — or a cross-project preference that belongs in your global CLAUDE.md, or a personal one that belongs in `CLAUDE.local.md`? |
| Rule-vs-memory? | Does this read as a deliberate, team-decided rule — or as something that reads more like a pattern Claude discovered on its own? |

Failing **True / Live / Consistent / Actionable** always routes to CHALLENGE — never silently fixed. Failing **only** Redundant-by-order routes to COMPRESS or CHALLENGE, never straight to DELETE. Failing **only** Correctly-scoped or Rule-vs-memory also routes to CHALLENGE — scope and intent are decisions only you can make.

## The five verdicts

| Verdict | Meaning | When it applies |
|---|---|---|
| **KEEP** | Behavior-changing rule, already terse | Passes the behavior test and all seven interrogation tests as-is |
| **COMPRESS** | Stays here, rewritten tighter | Same meaning in fewer lines, or fails only Redundant-by-order and trimming to a one-line acknowledgment preserves real project-specific nuance |
| **RELOCATE** | Moves to its natural home; a rich pointer (the concrete fact, not just "see docs/x.md") stays behind | A procedure that belongs in a skill or `docs/`, or a fact scoped to part of the codebase that belongs in a path-scoped `.claude/rules/` file |
| **DELETE** | Removed outright | Only when grep-confirmed: a verbatim duplicate elsewhere (location cited), or a dead reference (the failed lookup cited) |
| **CHALLENGE** | Raised to you as a question, never resolved unilaterally | Anything the grep can't settle either way — ambiguous liveness, a contradiction, vague wording, an unresolved encryption/CI flag, or a scope/intent question |

CHALLENGE follows the **lean CHALLENGE principle**: it's reserved for questions only you can answer. Anything a grep, a doc check, or another verification pass could settle gets resolved automatically, iterating until it's actually settled — never punted to you for convenience. When it does fire, CHALLENGE items in the plan are grouped by stakes: destructive/safety-relevant first, then widely-depended-on rules, then everything else batched as minor items you can wave through together.

## Preflight checks

Before touching anything, every run works through:

1. **Version-currency check** — compares the running session's own skill version against what's actually installed. A mismatch means the session is executing stale cached instructions; it stops with restart guidance (`/reload-plugins` alone is not enough — the session needs a full restart).
2. **Git repo check** — confirms you're in a git repo; if not, asks which file to tidy and skips the remaining git steps.
3. **Sync** — `git fetch`, checks branch/ahead-behind/dirty state.
4. **Visibility check** — public or unknown-visibility repos activate the global PRIMARY CHECK for every file this skill writes.
5. **Encryption check** — git-crypt/SOPS/age references force any unlock instructions to KEEP (never RELOCATE, to avoid a lock-out) and gate other relocations on matching encryption scope.
6. **CI-dependency check** — if a CI script greps the target file's content, that content can't be RELOCATEd without your explicit confirmation.
7. **User-level versioning check** — if the run might write under `~/.claude`, offers (never forces) a one-time bootstrap: a git repo there with an ignore-all `.gitignore` that tracks only instruction files, so `~/.claude` edits get history and reverts while credentials, transcripts, memory, and plugins stay untracked by default.

## Applying changes

**The reversibility gate.** After the plan is shown, what happens next depends on whether the target is protected:

- **Git-tracked, or memory with a fresh snapshot** → applied autonomously, one commit per iteration, with the autonomy explicitly announced.
- **Unversioned and unprotected** → waits for your confirmation; you can approve all, approve partially, or amend verdicts.
- **Always waits, regardless of tier**: every CHALLENGE, every write outside the repo being tidied (shown individually — never bundled into a bulk approval), and anything the PRIMARY CHECK flags.

**Order of operations.** Branching conventions are re-checked immediately before the first commit — even for a one-line edit, even when the file being committed is itself the source of those conventions. RELOCATEs happen first (with two-way cross-links), then COMPRESS, then DELETE. Every moved or renamed term gets grepped and its references updated in the same change.

**No-loss check and self-review.** Every removed line must be traceable to its new home or to a verified DELETE. Then a fresh, adversarial self-review pass hunts for information loss, broken references, or contradictions the edits themselves introduced — fixing and re-reviewing until a pass comes back clean, one commit per iteration.

## Run logs

Every run — even a report-only or aborted one — writes its own permanent file to `${CLAUDE_PLUGIN_DATA}/RUNS-archive/`. Nothing is ever shared across runs, and nothing is ever deleted.

**Filename:** `<YYYYMMDD>-<HHMM>-<target>.log`, local date/time — e.g. `20260708-2327-job2026.log`. `<target>` is `user` for a run scoped to `~/.claude`, or the target repo's directory name otherwise.

**Contents:** the target's absolute path, the commit(s) the run produced (or "n/a" if none), which target classes were covered, a memory-snapshot path if one was taken, and the full result — what changed, which instructions fired, user feedback, questions asked, friction, anything left uncovered.

These logs are the training data for the reflect skill.

## The reflect skill

`/tidyclaudemd:claudemd-tidy-reflect` learns from real runs and improves `claudemd-tidy` itself. Its one rule: **no evidence, no change** — it never invents an improvement from first principles.

**Where evidence comes from:**
- Every run log not yet marked `Processed: yes`.
- The live experience of a tidy run that just happened in this session.
- A dry run against a specific CLAUDE.md path you give it.
- An ad-hoc lesson you raise in conversation (the conversation itself stands in as the citation).
- Whether new evidence corroborates or contradicts an existing provisional lesson.

**What counts as a signal**, strongest first: feedback a run already tagged `[general]` or `[general?]`; a verdict you overrode; a question the skill had to ask that the instructions should have answered; friction during apply; content that fit no verdict or destination; an instruction that never fired across every log that covered its target class.

**Before a lesson is applied**, it must pass three tests: **generalizable** (helps other repos too, not just this one), **traceable** (cites the specific log or instance behind it), and **correctly homed**:

| Kind of lesson | Goes to |
|---|---|
| Tidy mechanics (scanning, verdicts, apply order, recording) | `claudemd-tidy/SKILL.md` |
| Reflection mechanics (this skill's own workflow) | `claudemd-tidy-reflect/SKILL.md` — the loop applies to itself |
| What counts as good CLAUDE.md content | The global hygiene rules in `~/.claude/CLAUDE.md` — proposed to you, never forked into the skill |
| A repo-specific quirk | That repo's own CLAUDE.md, suggested to you — never folded into the general skill |

**Provisional lessons.** A lesson drawn from a single log is applied but marked provisional. A second, independent run that corroborates it promotes the lesson (the tag is dropped, both logs cited); a contradicting run demotes or revisits it. This keeps one-off feedback from getting baked permanently into a suite meant to work across arbitrary repos.

**Pruning.** Reflection isn't only additive — an instruction that never fired across every log covering its target class is a removal candidate, with the same evidence discipline as an addition.

**Bookkeeping every pass:** applies minimal diffs, bumps the suite version, writes a CHANGELOG entry citing the log filename, updates the relevant README bullet (paraphrased, never copied verbatim), and marks each consumed log `Processed: yes (vX.Y.Z)` — or `, provisional` — in place.

## Invariants

Self-improvement can never remove or weaken these autonomously — any lesson touching one is presented to you with pros and cons, applied only on your explicit approval.

1. **Reversibility gate** — autonomous edits only when every change is individually revertible; unversioned or unprotected files always wait for your approval; intent, preference, scope, and out-of-repo writes always stop for you, regardless of tier. *(Rewrote the former stop-and-confirm invariant, with explicit approval, 2026-07-08.)*
2. **No-loss guarantee** — slimming relocates content; nothing is ever lost.
3. **Evidence before DELETE** — every deletion cites a located duplicate or a verified-gone reference.
4. **PRIMARY CHECK** — public or unknown-visibility repos get the sensitive-content gate on every file this skill writes.
5. **Single-sourced rules at runtime** — the skill reads hygiene rules only from your global `~/.claude/CLAUDE.md`, never a private copy. *(The one-time bootstrap template is a seed, not an exception — user decision, 2026-07-03.)*
6. **No evidence, no change** — the reflect skill never improvises.

## Optional companion hook

`claudemd-tidy` only runs when you invoke it, so nothing notices a CLAUDE.md crossing the ~150-line hygiene guardrail *between* runs. A deterministic hard cap would fight against the judgment-driven verdicts above, so instead here's an optional, purely advisory `PostToolUse` hook that just reminds you:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "f=\"$CLAUDE_TOOL_INPUT_FILE_PATH\"; case \"$f\" in *CLAUDE.md) n=$(wc -l < \"$f\"); if [ \"$n\" -gt 150 ]; then echo \"$f is now $n lines (> the ~150-line hygiene guardrail) — consider running /tidyclaudemd:claudemd-tidy\" >&2; fi ;; esac"
          }
        ]
      }
    ]
  }
}
```

This is guidance, not a shipped part of either skill — install it only if you want the reminder; nothing in the suite depends on it.
