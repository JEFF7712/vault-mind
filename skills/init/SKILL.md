---
name: init
description: Use to set up (or re-profile) a vault for the vault-mind skills, e.g. "set up vault-mind", "init my vault", "vault-mind isn't finding my notes", "re-detect my vault structure". Inspects the user's Markdown vault, infers its layout and frontmatter schema, confirms with the user, and writes a vault profile that every other vault-mind skill reads. Run this once before using quiz/feynman/ask/etc. Writes only the profile file, never touches note content.
---

# Init (profile the vault)

Every other vault-mind skill (`quiz`, `feynman`, `discuss`, `socratic-philosophy`, `ask`,
`connect`, `gaps`, `lint`, `recommend`, `fetch`) needs to know **this user's** vault shape:
where notes live, where readings live, what the frontmatter keys are called, what `status`
and `rating` mean here. Vaults differ. This skill detects that once and writes it to a
**profile** the others load, so the rest of the kit is portable instead of hardcoded.

**What this writes:** only `<vault>/.vault-mind/profile.md` (and creates `.vault-mind/`).
It never reads-then-rewrites the user's note content. It may *read* notes to infer schema.

## When to run

- First use of vault-mind in a vault (no `.vault-mind/profile.md` yet).
- The user reorganized their vault (renamed folders, changed frontmatter keys).
- A skill reports it can't find the profile or the layout looks wrong.

## Procedure

### 1. Locate the vault root

The vault root is the directory the user runs Claude from, unless they say otherwise. If
there's an `.obsidian/` directory, that marks an Obsidian vault root, use it. Confirm the
root with the user if ambiguous (e.g. they're in a parent dir containing several vaults).

### 2. Detect the layout (don't assume; look)

Inspect the actual tree, then infer. Useful probes (from the vault root):

```bash
fd -t d -d 2 .                                   # top-level folders
fd -e md . -d 2 | head -50                        # where .md files cluster
# sample frontmatter to learn the keys actually used:
fd -e md . -d 3 -x sh -c 'sed -n "/^---$/,/^---$/p" "$1" | head -20' _ {} \; 2>/dev/null | sort | uniq -c | sort -rn | head -40
```

Infer and record:
- **notes_dir**: where atomic/concept notes live (often `Notes/`, `Zettel/`, `Slipbox/`, or the root).
- **resources_dirs**: where reading material lives, with the `class` each maps to
  (e.g. `Resources/Books`→Book, `Resources/Papers`→Paper, `Sources/`→generic). May be empty
  if the user keeps no reading log, several skills degrade gracefully without it.
- **assets_dir**: where images / fetched sources go (for `fetch`).
- **templates_dir**: if present.

### 3. Detect the frontmatter schema

From the sampled frontmatter, determine the **actual key names and value vocabularies** this
vault uses; do not impose vault-mind's defaults:

- The **note** keys: tag key (`tags`?), links key (`related`?), source key (`resources`?),
  date key (`date`/`created`?), and whether a `reviewed`/review-date key exists (quiz needs
  one for spacing, if absent, propose adding `reviewed`).
- The **reading** keys: title, author, a **status** key + its value set (e.g.
  `read|reading|to be read|DNF`, or `done|todo`, or Dataview `#status/...` tags), and a
  **rating** key + its scale (0–10? 1–5 stars? none?).
- The **wikilink style**: `[[Title]]` vs `[[path/Title]]` vs none (plain MD links).
- Tag style: hierarchical (`a/b`) or flat.

When the vault is inconsistent or sparse, **ask the user** rather than guessing, a wrong
schema makes every downstream skill grade/scope incorrectly.

### 4. Confirm, then write the profile

Show the user the inferred profile and have them correct it. Then write
`<vault>/.vault-mind/profile.md` using `assets/profile-template.md` as the structure, filled
with real values. Also create `.vault-mind/` if needed. If the vault uses git, mention they
can `.gitignore` `.vault-mind/` (it's local config) or commit it (to share the profile) ,
their call.

If the chosen note schema lacks a review-date key and the user wants spaced `quiz`, add a
`reviewed` key to the profile's note schema and tell them `quiz` will stamp it on demand.

### 5. Hand off

Tell the user the profile is written and the kit is ready, list the available skills
(`/vault-mind:quiz`, `:feynman`, `:discuss`, `:socratic-philosophy`, `:ask`, `:connect`,
`:gaps`, `:lint`, `:recommend`, `:fetch`), and point at the natural loop:
`recommend → (read) → discuss / socratic-philosophy → you write the note → feynman → quiz → connect`.

## The kit's core rule (state it to the user)

vault-mind never writes the user's note **content**: that's where the learning lives. Skills
read, quiz, surface, and at most edit **metadata/links** (with approval). The user does the
thinking and the writing. If they want an AI that drafts notes for them, this is the wrong kit.
