---
name: lint
description: Use when the user wants to check or clean up the vault's consistency/hygiene, e.g. "lint my vault", "find broken links", "any duplicate or empty notes", "check my frontmatter", "audit my tags". Reports structural/metadata problems (broken links, dupes, empty notes, frontmatter drift, tag-taxonomy drift) and fixes the safe ones with approval. Never writes note prose.
---

# Lint (vault hygiene)

Find and (with approval) fix **structural and metadata** problems in the vault, broken
links, duplicate/empty notes, frontmatter drift. These are mechanical issues, distinct from
the *content* of notes.

**Core rule: metadata/structure edits are fine with approval; never write note prose.** Fixing
a broken wikilink or a status typo is fair game. Merging two notes' *content*, or filling an
empty note's body, is not, propose it and let the user do the writing.

Read the vault profile (`.vault-mind/profile.md`) first if you haven't this session, the
schema in it is the spec you're linting against (run `/vault-mind:init` if missing). Run
checks from the vault root; substitute the profile's dir/key names below.

## Checks

Run these, collect findings, present grouped by severity.

### 1. Broken / case-mismatch wikilinks
Links that don't resolve to a note or resource file (and aren't embeds/people).
```bash
rg -oN '\[\[[^]|#]+' <notes_dir> | sed 's/.*\[\[//; s/^ *//; s/ *$//' \
  | rg -vi '\.(png|jpe?g|gif|webp|avif|svg|mp4|pdf|excalidraw)$' \
  | rg -v '^[0-9 ,.]+$' | sort -u > /tmp/linked.txt
fd -e md . <notes_dir> <resources_dirs> | sed 's#.*/##;s#\.md$##' | sort -u > /tmp/existing.txt
comm -23 /tmp/linked.txt /tmp/existing.txt
```
For each unresolved target, case-insensitively re-check against `/tmp/existing.txt` (`rg -ix`)
and check the aliases key (if any). A **case/whitespace match to an existing note = a broken
link to fix** (e.g. `koopman operator` → `Koopman Operator`, or a stray leading space). A
target that matches nothing real is a `/vault-mind:gaps` item, not a lint fix.

### 2. Duplicate / near-duplicate notes
Exact basename collisions, and notes that are identical after stripping a trailing
parenthetical (a common near-dup pattern).
```bash
fd -e md . <notes_dir> | sed 's#.*/##;s#\.md$##' \
  | awk '{o=$0; gsub(/ *\([^)]*\)$/,""); print tolower($0)"\t"o}' \
  | sort | awk -F'\t' '{if($1==p)print "  "pl"  ~  "$2; p=$1; pl=$2}'
```
Merging is a *content* decision, propose which to keep and let them merge/write.

### 3. Empty-body notes (frontmatter only)
```bash
for f in <notes_dir>/*.md; do
  body=$(awk 'n==2{print} /^---$/{n++}' "$f" | tr -d '[:space:]')
  [ -z "$body" ] && echo "  EMPTY BODY: $f"
done
```
Report them; they write the body (or delete the stray).

### 4. Frontmatter drift (against the profile schema)
- **status** ∈ the profile's status value set, flag anything else (typos, blanks on items
  that should have one). `rg --no-filename --no-line-number '^<status_key>:' <resources_dirs> | sort | uniq -c`
- **rating** within the profile's scale (or blank/DNF), flag out-of-range / wrong-format values.
- **Missing required keys** for the `class`: readings need title/author/status; notes need
  tags/class. Flag missing/empty ones.
- **Tags**: flag obviously malformed tags (spaces, stray punctuation) and inconsistent
  spellings/depths. Don't force a taxonomy.

## Presenting and fixing

1. Present findings grouped: **broken links**, **dupes**, **empty notes**, **frontmatter**.
   Lead with the safely-fixable ones.
2. Offer to fix the **mechanical** ones, broken/case-mismatch links, status typos,
   malformed frontmatter values, key ordering. Apply only what they approve; preserve
   frontmatter key order; confirm each change.
3. For **content decisions**: merging near-dupes, writing an empty note's body, propose
   the action but **don't do the writing**. Deleting a note (even a stray) is destructive:
   ask before removing.
4. Don't touch editor config, saved-view, template, or asset dirs unless asked.
