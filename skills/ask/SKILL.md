---
name: ask
description: Use when the user asks a question about the contents of their own vault — e.g. "what did I conclude about functionalism", "what do my notes say about lag time in MSMs", "which books did I rate above 8", "summarize what I know about Koopman operators". Answers strictly from their notes/readings, cited to the source files. Read-only — never writes.
---

# Ask (grounded Q&A over the vault)

Answer the user's questions about *their own* vault — what they've written, read, rated,
concluded. The answer must come from the vault and be **cited to the notes/readings it came
from**, so they can trust and trace it. This is retrieval, not teaching and not opinion.

**Core rule: read-only.** Writes nothing.

Read the vault profile (`.vault-mind/profile.md`) first if you haven't this session (paths +
frontmatter schema; run `/vault-mind:init` if missing). Below, "notes dir", "resources dirs",
"status/rating keys" refer to the profile's values.

## Procedure

### 1. Find the relevant material

Search broadly before answering — names, tags, wikilinks, frontmatter fields.

```bash
# from the vault root; substitute the profile's dir/key names:
rg -l -i 'term' <notes_dir> <resources_dirs>            # files on the topic
rg -i 'term' "<notes_dir>/X.md"                          # what a note actually says
rg -l '<status_key>: <read_value>' <resources_dirs>      # consumed readings
rg -N '^(<rating_key>|<status_key>|tags|genres):' <resources_dirs>/**/*.md   # metadata queries
```

For **metadata questions** (ratings, status, counts, "what have I read on X"), read
frontmatter across the relevant folder rather than guessing.

### 2. Answer from the vault, with citations

- Base the answer **only on what's in the vault**. Quote or paraphrase their notes; cite the
  source as a wikilink or path (e.g. "per [[Some Note]]" or its file path).
- If several notes bear on it, synthesize across them and cite each.
- Distinguish clearly between **what their notes say** and any **outside knowledge** you add.
  If they only want what's in the vault, keep outside knowledge out unless they ask.
- **If the vault doesn't answer it, say so** — don't pad with general knowledge as if it
  were theirs. Offer to widen the search, or note it's a `/vault-mind:gaps` candidate
  (something they haven't written up) or a `/vault-mind:recommend` topic (something to read).
- Flag if their notes are **thin, stale, or internally inconsistent** on the question — that's
  useful signal, and a cue they may want to revise (themselves).

### 3. Stay read-only

Never create or edit files. If answering reveals a gap or error, report it and point to the
right skill (`/vault-mind:gaps`, `:connect`, `:lint`, `:feynman`) — they act on it.
