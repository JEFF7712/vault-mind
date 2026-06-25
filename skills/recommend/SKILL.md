---
name: recommend
description: Use when the user asks for reading recommendations (books, essays, papers, videos) on a topic, e.g. "recommend readings on philosophy of mind", "what should I read next about X", "find me something on Y". Profiles their vault (what they've read, rated, and taken notes on), finds gaps and connections, then suggests specific, current readings grounded in web search.
---

# Recommend readings

Recommend readings on a topic, grounded in what's already in the user's vault. The point is
**personalized** recommendations: tied to concrete evidence of what they've read, how they
rated it, and what concepts they've mapped, not generic "best of" lists. Recommendations
must be **specific and real** (actual titles/authors/URLs), verified by web search, never
invented from memory.

Read the vault profile (`.vault-mind/profile.md`) first if you haven't this session, it
defines the schema and what status/rating/tags mean here (run `/vault-mind:init` if missing).
"Notes dir", "resources dirs", "status/rating keys" below refer to the profile's values.

## Procedure

Work through these in order. Use TodoWrite to track them.

### 1. Profile them on the topic

Build a picture from the vault. Search broadly: the topic's exact tag, related tags
(if tags are hierarchical, `a/b` ⊂ `a`), and concept names.

- **Readings consumed**: resources with status read (or reading) in/near the topic. Note
  rating and genres. These define their taste and what they already cover.
- **Backlog**: status `to be read` (or the profile's equivalent), already on their radar;
  don't re-recommend, but it signals direction. Mention if a strong rec is already in backlog.
- **Negative signal**: DNF/abandoned items, note what they bounced off.
- **Concept map**: notes with matching tags/names. A dense note cluster = real engagement
  even with few formal readings. Follow wikilinks and the links/source keys to map the
  neighborhood and find where they read things.

```bash
# from the vault root; substitute the profile's dir/key names:
rg -l 'topic-tag' <resources_dirs> <notes_dir>          # files touching the topic
rg -N '^(<status_key>|<rating_key>|tags|genres):' <resources_dirs>/<file>   # one item's signal
rg -l '^<status_key>: <read_value>$' <resources_dirs>    # everything consumed (anchor $, "read" is a prefix of "reading")
```

### 2. Infer taste and find gaps

Synthesize, don't just list:

- **Taste vector**: which authors/genres/themes they rate highly. Primary texts vs. surveys?
  Fiction that explores the idea vs. nonfiction argument?
- **Gaps worth filling**: concepts they have notes on but no reading; canonical/foundational
  works missing from a cluster they care about; an adjacent subtopic they keep circling;
  under-covered formats (check whether a format like papers or videos is thin).
- **Connections**: a reading that bridges two clusters they're into.

### 3. Web search for specific readings

Search the web (include the current year for recency where it matters) to find **real,
specific** readings that fill the gaps and fit their taste. Prefer things genuinely adjacent
to what they already value over famous-but-generic picks. Mix formats when sensible.

**Verify every recommendation before presenting it**: search snippets routinely point at the
wrong thing (a *comment on* a paper, a summary, a paywalled stub, a dead link). For each pick:

1. Confirm the work is real with the correct **title + author**.
2. Pick a URL and **`WebFetch` it** to confirm it resolves to *that* work, not a review,
   commentary, citation list, login wall, or 404. Ask e.g. "Is this the full text / official
   page of '<title>' by <author>? Answer yes/no and what it actually is."
3. If it isn't the right target, pick another URL and re-fetch. Prefer: author's own site →
   official publisher / journal / arXiv/PhilArchive → a stable PDF → a reputable archive.
   Avoid scribd/coursehero/studocu and similar.
4. If after a couple of tries you can't confirm a clean link, still recommend the work but
   say the link is unverified (or give only the citation), never present a guessed URL as
   confirmed. Never invent a URL.

### 4. Present ranked recommendations

5–8 recommendations, ranked by fit. For each:

- **Title, Author** (format: essay/book/paper/video), with a link.
- **Why it fits you**: cite *specific vault evidence*, "you rated X highly and have notes on
  [[Y]], so…". No generic praise.
- **What gap/connection it fills.**
- A one-line honest caveat where relevant (e.g. "dense; you DNF'd a similar one").

Group by theme if that's clearer. Be honest if the vault is thin on the topic; say so and
lean more on taste + web search.

### 5. Offer to save (opt-in)

After presenting, ask whether to create `to be read` stub notes for any picks. Only if they
say yes:

- Create the file in the matching resources dir using the profile's reading schema (and a
  template from the templates dir if one exists).
- Fill what you know (title, author, publish, url, tags, genres); leave rating blank; set
  status to the backlog value. Keep template key order. Don't fabricate fields you don't know.

## Notes

- Don't recommend things already in the vault as read or to-be-read (cross-check first).
- Ground every "why" in real vault data. If you can't, say the rec is taste-/web-based, not vault-based.
- Never invent titles, authors, or URLs. Verify the work via search and the link via `WebFetch` (step 3).
