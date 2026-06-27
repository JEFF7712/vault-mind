---
name: audit
description: Use when the user wants their notes on objective subject matter (math, physics, CS, philosophy, etc.) checked for things that are actually wrong, e.g. "audit my MSM notes for errors", "fact-check my philosophy of mind notes", "is anything in my #physics notes incorrect", "audit what I've changed since last time". Batch-scans standing note CONTENT for factual errors, internal contradictions, misattributions, and conceptual conflations, web-verifying suspect claims before flagging. Diagnoses; you fix. Never writes note prose; by default stamps a hidden audited-date HTML comment at the end of each checked note (kept out of Obsidian's Properties panel).
---

# Audit (correctness pass)

Sweep the *content* of objective-subject notes for claims that are actually **wrong**, and
hand back a confidence-gated findings list. This is the passive correctness layer: `feynman`
and `quiz` only catch errors in notes you show up to engage with live; audit checks the notes
you'd never think to re-open.

**Core rules:**
- **Never write note prose.** You diagnose: name the wrong claim, the correct version, and a
  source. The user repairs their own note. The only write you ever make is the `audited` stamp:
  a hidden `<!-- audited: YYYY-MM-DD -->` HTML comment appended at the **end of the note**, which
  you do by default (never prose); the user can opt out by saying so. It lives in a trailing
  comment, not a frontmatter key, deliberately, so it stays out of Obsidian's Properties panel.
- **Objective matter only.** Math, physics, CS, philosophy, etc. — claims with a defensible
  right answer. Skip personal/journal/opinion notes; if a target looks personal, say so and
  leave it alone.
- **Web-verify suspect claims; never assert from memory.** For any factual or attribution
  claim you suspect is wrong, search to confirm *before* surfacing it. If you can't confirm,
  drop it or mark it low-confidence — your own recall is not a source.
- **Every finding carries `claim → correction → source`.** No triple, no surface. "This looks
  wrong" without the correct version and a citation is noise; suppress it.

Read the vault profile (`.vault-mind/profile.md`) first if you haven't this session (paths +
frontmatter schema); run `/vault-mind:init` if missing. Run from the vault root and substitute
the profile's dir/key names below. The `audited` stamp is a trailing HTML comment, not a
frontmatter key (see step 6); the profile documents it as such.

## 1. Resolve the target set

From the user's prompt, build the list of notes to audit:

- **Topic** ("my MSM notes", "philosophy of mind") → match notes by title/tags/body in
  `<notes_dir>`. When in doubt, list the matched notes and confirm scope before spending tokens.
- **Tag** (`#physics`, `area/ml`) → notes carrying that tag (respect the profile's tag style).
  ```bash
  rg -lN '^\s*-?\s*#?<tag>\b' <notes_dir>
  ```
- **Folder** → `fd -e md . <folder>`.
- **Bare invocation / "since last time"** → incremental mode (next step).

Pull in each target's wikilinked neighbors as *context* (so cross-note contradictions are
visible) even though you only report on the target set itself.

## 2. Incremental skip

Audit stamps a trailing `<!-- audited: <date> -->` comment (see step 6). Skip any note already
audited at or after its last edit, it hasn't changed since you last checked it. In bare/"since
last time" mode this *is* the selector: audit every note whose audited stamp is missing or older
than its file mtime.

```bash
# notes changed since their last audit stamp (mtime newer than the stamp, or no stamp)
for f in <notes_dir>/*.md; do
  stamp=$(rg -oNm1 'audited:\s*\K[0-9]{4}-[0-9]{2}-[0-9]{2}' "$f")
  mtime=$(date -r "$f" +%F)
  { [ -z "$stamp" ] || [ "$stamp" \< "$mtime" ]; } && echo "$f"
done
```

Tell the user how many notes are in scope and roughly the cost before grinding through a large
set; offer to narrow. Web verification is the expensive part, so the skip matters.

## 3. Load ground truth per note

For each target note: read the note **and the source readings it cites** (the profile's
source-readings key, e.g. `resources`) so you're checking against what the note was built from,
not just your own model. This is the same grounding move as `/vault-mind:feynman`.

## 4. Extract claims and sort candidate issues

Pull the note's checkable assertions (definitions, formulas, named results, attributions,
causal/logical claims). Sort candidate problems into buckets, **priority-ordered**:

1. **internal inconsistency** — the note contradicts itself, or contradicts another note in
   scope/its neighbors. Highest signal, needs no web (it's pure cross-reference of *your* corpus).
   Cite both sides.
2. **factual / notational error** — wrong formula, sign, definition, constant, units, a flipped
   inequality, an off-by-one in an algorithm's complexity, etc.
3. **misattribution** — wrong person, date, paper, or who-said-what / who-proved-what.
4. **conceptual conflation** — two distinct ideas merged as if one (e.g. the Koopman operator
   vs. its generator; type vs. token; necessary vs. sufficient).
5. **interpretive pushback** *(separate, low-priority)* — not an error: an oversimplification,
   a missing caveat, or a claim you'd argue with. Walled off from the above so it never
   masquerades as "wrong."

## 5. Verify before surfacing

- For every candidate in buckets 2–4, **web-search to confirm** it's actually wrong and to get
  the correct version. Prefer authoritative sources (textbooks, primary papers, SEP for
  philosophy, official docs). Confirmed → keep, with the source. Can't confirm the note is
  wrong → drop it or downgrade to low confidence; do not flag on suspicion alone.
- Bucket 1 (inconsistency) is verified by quoting the two conflicting passages, no web needed,
  but if resolving *which* side is right needs a fact, verify that fact.
- **Confidence-gate.** Tag each survivor high / medium / low. Show high + medium by default;
  hold low-confidence ones back unless the user asks for the long tail.
- Watch for false positives that plague advanced material: deliberate simplifications, valid
  shorthand, context-dependent statements, and correct-but-unusual conventions are **not**
  errors. When the note is defensible under a reasonable reading, don't flag it.

## 6. Present, then stamp

1. Present findings **grouped by note**, errors first (buckets 1–4 in priority order) and the
   interpretive-pushback bucket last and visibly separate. For each finding give:
   `file:line` · bucket · confidence · **the wrong claim** → **the correct version** → **source**.
2. If a note came back clean, say so plainly, that's a real result, not a non-answer.
3. **By default**, stamp every note you actually checked (clean or flagged), no need to ask
   first. Skip the stamp only if the user opted out. The stamp is a single hidden line appended
   at the **end of the note**:
   ```
   <!-- audited: <today> -->
   ```
   If the note already has one, update its date in place rather than adding a second. Use an HTML
   comment, not a frontmatter key, on purpose: it stays out of Obsidian's Properties panel and is
   invisible in reading/live-preview. This is the only write. **Do not edit note prose**, even to
   "just fix" an obvious typo in a claim; that's the user's to fix (and mechanical frontmatter
   issues are `/vault-mind:lint`'s job).
4. Offer to re-audit a note once they've revised it.

## Scope guards

- Don't audit personal, journal, daily, or opinion notes; if the target set sweeps some in, name
  them and exclude them.
- Don't touch editor config, templates, saved-view, or asset files.
- Don't rewrite, merge, or delete notes. Findings in, metadata stamp out.
