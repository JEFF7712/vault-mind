---
name: gaps
description: Use when the user wants to find concepts they should write notes for but haven't — e.g. "what notes am I missing on reinforcement learning", "find gaps in my philosophy notes", "what should I write next". Surfaces terms/concepts referenced across their notes and readings that have no note yet, ranked as a next-write queue. Never writes the notes for them.
---

# Gaps (next-write queue)

Find the concepts the user keeps *using* but hasn't *written up* — the holes in their
Zettelkasten. The output is a prioritized list of notes **they** should write next. This skill
never writes those notes; the writing is the learning.

**Core rule: never author note content.** This skill only reads and reports. It does not
create note files (not even stubs — concept notes are theirs to write; that's the whole point).

Read the vault profile (`.vault-mind/profile.md`) first if you haven't this session (paths +
frontmatter schema; run `/vault-mind:init` if missing). "Notes dir", "resources dirs" below
refer to the profile's values.

## Procedure

### 1. Scope

- **Topic** (`gaps in my RL notes`): restrict to that cluster (by tag/name).
- **Whole vault** (`what should I write next`): scan the notes dir broadly.

### 2. Find the gaps

Three signals, strongest first:

1. **Dead wikilinks** — `[[Concept]]` references pointing to notes that don't exist. These
   are gaps they *already flagged themselves* by linking to them.
   ```bash
   # from the vault root; substitute the profile's dir names. links FROM note bodies;
   # strip embed/asset and math-matrix noise, trim whitespace:
   rg -oN '\[\[[^]|#]+' <notes_dir> | sed 's/.*\[\[//; s/^ *//; s/ *$//' \
     | rg -vi '\.(png|jpe?g|gif|webp|avif|svg|mp4|pdf|excalidraw)$' \
     | rg -v '^[0-9 ,.]+$' | sort -u > /tmp/linked.txt
   # "exists" must include BOTH notes and resources dirs (source links point into resources):
   fd -e md . <notes_dir> <resources_dirs> | sed 's#.*/##;s#\.md$##' | sort -u > /tmp/existing.txt
   comm -23 /tmp/linked.txt /tmp/existing.txt    # linked but no note
   ```
   Filter the result before reporting:
   - **Case/whitespace mismatches to an existing note are NOT gaps — they're broken links**
     (e.g. `koopman operator` → existing `Koopman Operator`). Route those to `/vault-mind:lint`,
     don't list them to write. Detect by case-insensitive re-check against `/tmp/existing.txt` (`rg -ix`).
   - Account for an aliases key (if the profile defines one) — a "missing" target may exist
     under an alias. Verify before reporting.
   - Scanning the notes dir only (not resources) keeps author/person links — which live in
     reading `author` frontmatter — out of the results. Authors aren't concept gaps.
2. **Recurring un-noted terms** — concepts named across multiple notes/readings (especially
   technical terms or names) that have no note and aren't even linked yet.
3. **Cluster holes** — a topic they clearly engage with where a foundational/connecting
   concept is conspicuously absent (e.g. a method whose prerequisites have notes but the
   method doesn't).

### 3. Present the queue

Rank by how central/recurring the concept is and how much it would knit existing notes
together. For each gap:
- **Concept name** — where it's referenced (which notes/readings, how many times).
- Why it's worth writing — what it would connect or unblock.
- For dead links: note that they already linked to it from `[[X]]`.

Be honest if the vault is well-covered on the topic.

### 4. Hand off

Suggest they write the top one or two themselves. If they want to *learn* an unfamiliar gap
concept before writing it, point them to an external from-scratch tutor skill (if installed).
After they write a note, `/vault-mind:connect` can wire it in. **Create no note files.**
