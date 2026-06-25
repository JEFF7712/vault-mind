---
name: fetch
description: Use when the user wants the actual source document for a reading pulled down locally so the agent can read it directly, e.g. "fetch the source for Epiphenomenal Qualia", "download the papers in my philosophy of mind list", "grab a local copy of the VAMPnets paper", "fetch everything I haven't downloaded yet". Resolves the reading's online-source URL, downloads a publicly available copy into the assets dir, and verifies it. Downloads source files only, never writes note prose.
---

# Fetch (download source documents locally)

For a reading in the resources dirs, download the actual source document (the PDF or article
behind its online-source URL) into the vault so the agent can `Read` it directly instead of
relying on web search snippets. This makes the primary text available for `/vault-mind:discuss`,
`:feynman`, `:quiz`, `:ask`, and for the user's own reading.

**What this skill writes:** source files only (downloaded PDFs/HTML) into the assets dir. It
writes **no note content**: downloading a paper is not authoring understanding. The core rule
still holds: never write the body of a concept note.

**Legality/ethics, only publicly available sources.** Fetch open-access copies: direct PDFs,
arXiv, PMC, PhilArchive/PhilPapers, author/university pages, journal open-access. **Never
bypass paywalls, logins, or DRM, and never use shadow libraries (sci-hub, libgen, z-lib).**
If only a paywalled copy exists, report it as unavailable and move on; don't fake it.

Read the vault profile (`.vault-mind/profile.md`) first if you haven't this session (paths +
schema; run `/vault-mind:init` if missing). "Assets dir", "resources dirs", "url key",
"local key" below refer to the profile's values.

## Where downloads go

- Into the assets dir (e.g. an `assets_dir/Papers/` or `assets_dir/Essays/` subfolder).
- Name the file to **match the source note's basename** so note ↔ source maps
  deterministically (e.g. a note `VAMPnets … - Andreas Mardt.md` →
  `<assets_dir>/Papers/VAMPnets … - Andreas Mardt.pdf`).
- Consider gitignoring the fetched-source paths; they're a local cache, usually not committed.
- Books are usually not freely available, skip unless the user points at a known public copy.
  Video sources (a URL to a video, not a document) are out of scope.

## Procedure

### 1. Resolve targets and their URLs

A specific note, a topic, or "all not-yet-downloaded". Pull the url key from frontmatter.

```bash
# from the vault root; substitute the profile's dir/key names:
rg -N --no-filename '^(title|<url_key>):' <resources_dirs>     # a topic / all readings + URLs
rg -N '^<url_key>:' "<resources_dir>/Some Reading.md"          # one note's url
```

If a target has no url, say so (it's a `/vault-mind:lint`/metadata fix, or needs a source
found first, offer to search the web for one, but don't invent a URL).

### 2. Skip anything already local

Don't re-download. Check the target path, and scan the dir for an older/short-named copy.

### 3. Download a public copy

Resolve to an open-access document, then download with `curl -L` (follow redirects).

- **Direct `.pdf` URL** → download as-is.
- **arXiv** (`arxiv.org/abs/ID`) → use `https://arxiv.org/pdf/ID`.
- **DOI / publisher landing page** → try open access in this order: arXiv, PubMed Central
  (PMC), the author's/university page, the journal's own open-access PDF. A closed-journal DOI
  often has a free preprint, search for it. If none is open, mark **unavailable** and skip.
- **HTML essay** → save the article. Prefer a PDF if the source offers one; otherwise save
  the page (`.html`). If a readability/markdown converter (`pandoc`, etc.) is available,
  convert to clean text; if not, keep the `.html` and note that `WebFetch <url>` reads it
  more cleanly than the raw file.

```bash
curl -L --fail --retry 2 -A "Mozilla/5.0" -o "<assets_dir>/Papers/<note-basename>.pdf" "<resolved-url>"
```

### 4. Verify the download (don't trust the extension)

A renamed login/captcha/404 page is worse than nothing. The **magic-byte check is the
reliable test** (`file` is not installed everywhere):

```bash
head -c 5 "<path>.pdf"        # PDFs start with %PDF-
ls -la  "<path>.pdf"          # non-trivial size, not a 2 KB error page
file    "<path>.pdf" 2>/dev/null   # "PDF document" if `file` exists
```

Then sanity-check it's the **right** work: confirm title/author match the note. Reading a PDF
back needs **poppler-utils** (`pdftotext` for text, or `pdftoppm`, which the `Read` tool uses
to render pages). If poppler is missing the bytes are still valid; flag that the user should
install it so PDFs are readable in-session.

If it's the wrong document or an error page, delete it and report failure; never leave a
misnamed file in place.

### 5. Record the local copy in frontmatter

On a verified download, set the note's local key to a wikilink to the file so the user can
open their local copy. This is a metadata edit, the only frontmatter `/vault-mind:fetch`
touches; never the body.

```yaml
<local_key>: "[[<assets_dir>/Papers/<note-basename>.pdf]]"
```

If the local key isn't in the profile/template, it's a custom key; mention you're adding it.
Leave it blank if the fetch failed.

### 6. Report

State, per target: fetched (with the local path the agent can now `Read`, local key set),
already-present, or unavailable (and why, paywalled, no public copy, dead link → a
`/vault-mind:lint` item).
