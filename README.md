# vault-mind

Learning-first skills for a Markdown knowledge vault (Obsidian, Foam, Zettelkasten). The kit
reads, quizzes, connects, and pressure-tests your understanding, but you write your own notes.
The AI never writes note content for you.

## Setup

Run `/vault-mind:init` once. It detects your vault's layout and frontmatter schema and writes
`.vault-mind/profile.md`, which every other skill reads. That profile is what lets the kit
work on any vault shape.

## Skills

| Skill | What it does |
|-------|--------------|
| `init` | Profile your vault for the other skills. |
| `ask` | Answer questions about your vault, cited to your notes. Read-only. |
| `recommend` | Reading suggestions from your taste and gaps, plus web search. |
| `fetch` | Download the public source behind a reading's URL. |
| `quiz` | Active recall from your notes: interleaved, confidence-checked, gap-classified, spaced. |
| `feynman` | You explain a concept back; it finds the holes. |
| `discuss` | Socratic partner on a reading, before you write the note. |
| `socratic-philosophy` | Drill one argument: reconstruct, attack, defend, consolidate. |
| `connect` | Surface missing wikilinks between notes (you approve each). |
| `gaps` | Your next-write queue: dead links, recurring un-noted terms. |
| `lint` | Vault hygiene: broken links, dupes, empty notes, frontmatter drift. |

Skills are namespaced `/vault-mind:<skill>`. Only metadata, or by opt-in your own words, ever
gets written; never AI-authored prose.

## The loop

```
recommend -> read -> discuss / socratic-philosophy -> you write the note
   -> feynman -> quiz -> connect -> gaps (what to write next)
```

## Install

```
/plugin marketplace add JEFF7712/vault-mind
/plugin install vault-mind@vault-mind
```

Then run `/vault-mind:init` in your vault. (Local dev: `claude --plugin-dir /path/to/vault-mind`.)

Assumes an atomic-note vault with YAML frontmatter and `[[wikilinks]]`; sparser vaults degrade
gracefully.

## License

MIT.
