---
name: connect
description: Use when the user wants to find non-obvious connections between their existing notes — e.g. "what connects to my Koopman operator note", "find links I'm missing in my philosophy of mind notes", "connect my ML and chemistry notes". Surfaces conceptual overlaps that aren't wikilinked yet and suggests where THEY might draw the link. May insert a wikilink they approve; never writes note prose.
---

# Connect (surface missing links)

A Zettelkasten pays off through its links. This skill finds conceptual overlaps between
notes the user has *already written* that aren't yet connected, and shows them where a link
belongs — so they make the connection (and the understanding that comes with it). You find
candidate links; they decide and learn from them.

**Core rule: never author note prose.** Inserting a wikilink into the links key or fixing a
link is a metadata/structure edit and is fine **with their approval**. Writing explanatory
sentences in a note body is not — that's theirs.

Read the vault profile (`.vault-mind/profile.md`) first if you haven't this session (paths +
frontmatter schema; run `/vault-mind:init` if missing). "Notes dir", "links key", "tags key"
below refer to the profile's values.

## Procedure

### 1. Pick the neighborhood

- **Single note** (`connect X`): read note `X`, its links/wikilinks, and notes that share
  its tags. That's the candidate pool.
- **Topic/area** (`connect my philosophy of mind notes`): gather notes by tag/name match.
- **Cross-domain** (`connect my ML and chemistry notes`): pull both clusters; the goal is
  bridges between them.

```bash
# from the vault root; substitute the profile's dir/key names:
rg -l 'tag-or-term' <notes_dir>                         # candidate notes
rg -o '\[\[[^]]+\]\]' "<notes_dir>/X.md"                # existing links out of X
```

### 2. Find links that should exist but don't

For each candidate pair, judge: do these share a mechanism, structure, method, or idea —
and is that link **not already** present (not in either note's links key or body)?

Prioritize:
- **Non-obvious** overlaps (same math under different names; a method reused across domains).
- **Cross-domain bridges** — the highest-value Zettelkasten links.
- **Orphans / under-linked notes** — notes with an empty links key and few inbound links.
- **Dead wikilinks** — links pointing to notes that don't exist (also a `/vault-mind:gaps` signal).

Skip links that are already there or are trivial/restating the tag.

### 3. Present, ranked

Give 3–8 suggested connections, strongest first. For each:
- **`[[Note A]]` ↔ `[[Note B]]`** — the specific shared idea, in one or two lines.
- **Why it's worth linking** / what understanding it unlocks.
- Frame it as a prompt for *them*: "you could note that both X and Y are ... — want to draw
  that link?" Don't write the explanation for them.

Also call out orphans and dead links separately.

### 4. Apply only what they approve (metadata only)

If they say yes to specific links, you may edit **metadata/structure**:
- Add `[[Note B]]` to `Note A`'s links key (and vice-versa), preserving frontmatter order.
- Fix a dead/renamed wikilink.

Do **not** add prose to note bodies explaining the connection — that's the part they write
to actually learn it. Confirm what you changed.
