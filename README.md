# vault-mind

**Learning-first skills for a Markdown knowledge vault (Obsidian, Foam, plain Zettelkasten).
The AI never writes your notes for you. It makes you think.**

Most AI-for-notes tools draft, summarize, and auto-file *for* you. vault-mind takes the
opposite stance, deliberately: **writing your notes is the thinking, and the thinking is the
point.** These skills read, quiz, surface connections, and pressure-test your understanding,
but the notes stay yours to write. (This mirrors the standing consensus in the Zettelkasten
community: use AI for everything *around* the notes, never for the writing itself.)

If you want an AI that writes your second brain for you, this is the wrong kit.

## What's in it

Run **`/vault-mind:init` once**. It inspects your vault, infers its layout and frontmatter
schema, confirms with you, and writes a profile (`.vault-mind/profile.md`) that every other
skill reads. That profile is what makes the kit portable across differently-shaped vaults.

| Skill | What it does | Writes? |
|-------|--------------|---------|
| `/vault-mind:init` | Profile your vault so the other skills know its shape. | the profile only |
| `/vault-mind:ask` | Answer questions about your own vault, cited to your notes. | no (read-only) |
| `/vault-mind:recommend` | Reading suggestions grounded in what you've read/rated/noted, plus web search. | opt-in stubs |
| `/vault-mind:fetch` | Download the public source behind a reading's URL into your assets dir. | source files plus 1 metadata key |
| `/vault-mind:quiz` | Active-recall questions from your notes: interleaved, confidence-checked, gap-classified, spaced via a review-date stamp. | 1 metadata key |
| `/vault-mind:feynman` | You explain a concept back; it finds the holes against your note and the literature. | no |
| `/vault-mind:discuss` | Socratic partner on a reading; draws *your* take out before you write the note. | no |
| `/vault-mind:socratic-philosophy` | The argument gym: drill one argument (reconstruct, attack, defend, consolidate). | opt-in: saves the argument as a note in *your* words |
| `/vault-mind:connect` | Surfaces missing wikilinks between existing notes; inserts one only if you approve. | metadata links (approved) |
| `/vault-mind:gaps` | Your next-write queue: dead wikilinks, recurring un-noted terms, cluster holes. | no |
| `/vault-mind:lint` | Vault hygiene: broken/case-mismatch links, dupes, empty notes, frontmatter drift. | metadata fixes (approved) |

**The only skills that write anything write metadata, or by explicit opt-in save *your own*
words.** Never AI-authored note prose.

## The loop it's built around

```
recommend -> (you read) -> discuss / socratic-philosophy -> YOU write the note
   -> feynman -> quiz -> connect -> (gaps shows what to write next)
```

Each skill hands off to the next. `quiz` and `feynman` share one gap vocabulary
(faded / misconception / never-learned / imprecise) so a weak spot routes to the right fix.

## Install

This is a Claude Code plugin. During development, load it directly:

```bash
claude --plugin-dir /path/to/vault-mind
```

Then run `/vault-mind:init` inside your vault. To distribute, publish via a plugin marketplace
(see the Claude Code plugin docs). Skills are namespaced `/vault-mind:<skill>`.

## Design notes

- **Portable, not hardcoded.** Skills reference logical roles (notes dir, status key, rating
  scale) that `init` resolves to your vault's actual paths and keys in the profile.
- **Assumes** an atomic-note vault with YAML frontmatter and `[[wikilinks]]`. Sparse or
  reading-log-free vaults degrade gracefully (the reading-oriented skills just have less to
  work with).
- **`quiz` spacing** uses a `reviewed`-role date it stamps itself, a minimal built-in spaced
  repetition. If you already run a dedicated spaced-repetition plugin, prefer that; this is
  for vaults without one.

## License

MIT.
