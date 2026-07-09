# Cross-skill overlap/conflict detection — plan

Status: **reviewed — ready to implement** (4 adversarial passes, final pass clean). Written 2026-07-09 against TidyClaudeMD v0.20.2. Implements idea #2 from `competitive-landscape-2026-07-03.md` ("Features worth prototyping"): the `--skills` class currently interrogates each `SKILL.md` in isolation; nothing catches two skills that duplicate, collide, or trigger on the same requests. Two independent projects built this (evidence it earns its cost) — their mechanics were read at source level on 2026-07-09:

- **claudoctor** (`src/lib/analyze.ts`): four mechanical tiers — *duplicates* (identical `contentHash` across ≥2 paths), *near-duplicates* (identical `bodyHash`, i.e. same body after stripping frontmatter, but different `contentHash` — "frontmatter drift"), *name conflicts* (same `name`, different content **and** different body), *overlap* (Jaccard similarity ≥0.5 on stopword-filtered `name + description` token sets; `--deep` also compares bodies). Reports wasted tokens as `tokens × (copies − 1)`.
- **pulser** (`src/rules/conflicts.ts`): extracts **quoted** trigger phrases from each skill's `description` and warns when the same phrase appears in ≥2 skills — "overlapping trigger keywords cause Claude to invoke the wrong skill."

## Goal

Extend `claudemd-tidy`'s `--skills` class with a **sweep-level** analysis pass: after the existing per-file interrogation, compare every pair of skills in the sweep for duplication, name collision, and trigger overlap — and route findings through the suite's existing verdict/evidence machinery, not a parallel one.

## Design

### The four checks (sweep-level pass)

**Placement:** the full spec lives in the **Target classes table's Skills row** (where all class-specific behavior already lives), as a named addition — "Sweep-level checks" — after the three existing per-file tests; Step 2 gets only a one-line pointer in its orientation-pass bullet ("for `--skills`, also run the Skills row's sweep-level checks across all skills found"). This keeps Step 2's numbered list class-agnostic, matching how the class table already owns `--memory`'s snapshot rule.

Run once per `--skills` sweep, over all skills in scope (project `.claude/skills/`; plus `~/.claude/skills/` when combined with `--user`; `--all` composes identically to `--skills --user` here, so cross-scope pairs are compared whenever both scopes are swept):

| Check | Mechanics | Evidence grade |
|---|---|---|
| **Duplicate?** | Byte-identical `SKILL.md` content at ≥2 paths — `shasum` over each file, group by hash | Mechanical, hash-confirmable |
| **Near-duplicate?** | Identical body but different frontmatter — `shasum` over content with the YAML frontmatter block stripped (`awk` past the second `---`), grouped; flag groups whose full-content hashes differ | Mechanical |
| **Name-conflict?** | Same frontmatter `name:` in ≥2 skills whose content differs. Claude Code already disambiguates *some* clash shapes (nested-directory skills get `<dir>:<name>` qualified names; plugin skills are namespaced), so a genuine conflict needs one of two shapes this check targets: (a) a frontmatter `name:` that differs from its directory basename and collides with another skill's name — note the per-file **Frontmatter-sane?** test already flags the name/dirname mismatch itself, so this check adds the *pairing* (who it collides with), not the mismatch; (b) the same name across scopes (project vs. user), where load precedence silently decides which one the model sees — a shadowing ambiguity worth surfacing even though nothing "breaks" | Mechanical |
| **Trigger-overlap?** | Two skills whose `description`s would plausibly fire on the same user request. Mechanical assist first (pulser-style: quoted phrases appearing in ≥2 descriptions; plus high word-overlap between descriptions — claudoctor's 0.5 Jaccard on stopword-filtered tokens is the calibration reference), then LLM judgment on the candidates — this suite runs *as* an LLM, so it can judge "would these two descriptions race for the same prompt?" directly instead of approximating with Jaccard the way claudoctor must | Judgment, mechanically pre-filtered |

All four are cheap: hashing is one `shasum` pass over files already inventoried by the sweep. The mechanical pre-filter for Trigger-overlap? does compare all description pairs — but descriptions are short strings already read during the sweep, so that comparison costs nothing meaningful; the *LLM judgment* step is what's restricted to pre-filtered candidates. A skill whose frontmatter is malformed (no parseable `name:`/`description:`) is skipped by the sweep checks with a note — the per-file **Frontmatter-sane?** test already owns flagging it; the sweep must not crash on it.

### Verdict routing (Step 3)

Consistent with the suite's existing philosophy — mechanical evidence never auto-deletes a *skill*, because removing a whole skill is a scope/intent decision, not a dedup line-edit:

- **Duplicate** → **CHALLENGE**, evidence-cited (both paths, matching hash, and the redundant copy's size in lines — the waste figure stays line-based for now, per the tokenization bullet below): "these two files are byte-identical — which is the canonical home?" Removing one is DELETE-grade *evidence*, but *which* copy survives is user-only. Follows the sweep-level CHALLENGE precedent set by the AGENTS.md visibility check (v0.11.0).
- **Near-duplicate** → **CHALLENGE**, citing the diff of the two frontmatters: "same procedure, two names/descriptions — intentional variant or drift?"
- **Name-conflict** → **CHALLENGE**, tier-1 stakes (it affects which skill actually runs when invoked).
- **Trigger-overlap** → **CHALLENGE**, minor tier unless one of the pair is safety-relevant: "these two descriptions plausibly fire on the same request — differentiate, merge, or intentional?"

The suite's own two skills (`claudemd-tidy`, `claudemd-tidy-reflect`) remain excluded as **edit targets**, but participate as **comparison references** — a third-party skill that duplicates one of them should be flagged, with the finding landing on the non-suite copy. Note the location implication: the suite's skills live at `${CLAUDE_PLUGIN_ROOT}/skills/`, *not* in the sweep's scan locations, so the sweep must explicitly read them from the plugin root for comparison purposes only — they are never hashed into the findings as removable copies. When both members of a flagged pair are third-party, the CHALLENGE covers the pair jointly (no "which file carries the finding" rule needed — the user's resolution names the survivor).

### Report mode

The three mechanical checks (duplicate/near-duplicate/name-conflict counts) join `--report`'s mechanical-checks tier when composed with `--skills` — they're scriptable and judgment-free, exactly what that tier is for. Trigger-overlap judgment stays out of report mode (it's judgment, and report mode skips interrogation by design).

### Run log

No format change needed: sweep-level findings go under **Instructions exercised** (e.g. "CHALLENGE ×2: sweep-level Duplicate? check") like any other verdict source, and CHALLENGE resolutions land in **User feedback** as usual.

## Files & bump

- `skills/claudemd-tidy/SKILL.md`: Target classes table, Skills row — the full "Sweep-level checks" spec (the four checks, mechanics, pre-filter rule, suite-skills-as-references rule), per the Placement decision above; Step 2 orientation-pass bullet — the one-line pointer only; Step 3 — routing note (sweep-level CHALLENGE tiers); Report mode — one clause adding the mechanical trio.
- `docs/reference.md`: Target classes → skills-class tests paragraph gains the sweep-level checks.
- `README.md`: the `--skills` fragment of the Scope bullet mentions cross-skill detection.
- `CHANGELOG.md` + version bump: **MINOR** (new capability), with git tag + GitHub release in the same pass per the v0.20.2 rule.

## Verification

Build a disposable fixture repo (scratchpad) with a skills directory containing: an exact duplicate pair, a near-duplicate pair (same body, different `name:`/`description:`), a name-conflict pair exercising shape (a) — the same frontmatter `name:` in two differently-named directories, different bodies — one trigger-overlap pair (distinct wording, same plausible trigger), and one clean control skill. Run `--skills` against it and confirm: all four planted findings surface as CHALLENGEs with correct evidence citations, the control skill produces none, and `--report --skills` surfaces exactly the three mechanical counts. Record the run log as normal — it doubles as the first real multi-skill-class evidence for the reflect loop.

## What this plan does not decide

- **Plugin/marketplace cache sweeping** (landscape idea: claudoctor scans `~/.claude/plugins/cache/`) — a different location set with different semantics (cached copies are *expected* duplicates of installed versions); separate plan if pursued.
- **Exact tokenization** for waste estimates — findings cite line/byte counts for now; upgrading to real token counts is landscape idea #5, orthogonal.
- **Cross-agent directories** (`~/.codex/`, `~/.cursor/rules/`) — landscape idea #4, out of scope here.
- **Auto-merge of near-duplicates** — claudoctor has it on its own roadmap; this plan deliberately stops at CHALLENGE, per the suite's user-only-decisions principle.

## Iteration log

- 2026-07-09 — preliminary draft (pass 0), pre-review.
- 2026-07-09 — pass 1: 1 HIGH (Name-conflict rationale corrected — Claude Code already disambiguates nested/plugin name clashes, so the check now targets the two real shapes: frontmatter-name≠dirname collisions and cross-scope shadowing, with the Frontmatter-sane? interplay spelled out), 2 MEDIUM (suite-skills-as-references now states they're read from `${CLAUDE_PLUGIN_ROOT}`, never hashed as removable; placement decided — full spec in the class table's Skills row, Step 2 gets a one-line pointer), 5 LOW (N²-prefilter wording, malformed-frontmatter skip rule, `--all` composition, line-based waste figure made consistent, hash- not grep-confirmable).
- 2026-07-09 — pass 2: 1 HIGH (Files & bump section still described the pre-pass-1, inverted placement — Step 2 carrying the full spec — contradicting the Placement decision; now matches it), 1 LOW (verification fixture's name-conflict pair now specifies shape (a) so it exercises a genuinely possible collision).
- 2026-07-09 — pass 3: 2 LOW (iteration log reordered chronologically; Trigger-overlap pre-filter given a calibration reference — claudoctor's 0.5 Jaccard).
