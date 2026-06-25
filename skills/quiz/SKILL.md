---
name: quiz
description: Use when the user wants to test themselves / be quizzed / do active recall on a topic or note in their vault, e.g. "quiz me on Markov state models", "test me on this topic", "quiz me on stuff I haven't reviewed lately". Generates active-recall questions FROM their own notes, asks them interactively (interleaved, with a confidence check), grades answers against the note, classifies each gap, and flags misconceptions. Stamps a review-date for spacing but never writes note prose.
---

# Quiz (active recall)

Test the user on their own notes via active recall, the point is *they* retrieve the answer,
which is what strengthens memory. You generate questions from their notes, ask them
**interactively (one at a time)**, and grade their answers against what the note says plus
correct domain knowledge.

**Core rule: never write note content.** The *only* thing this skill writes is the review-date
frontmatter stamp (the `reviewed`-role key from the profile, a metadata edit, like
`/vault-mind:connect`'s links or `/vault-mind:fetch`'s local key), never note prose. Don't
offer to "save a summary" or fix notes; if a note is wrong/thin, tell the user so *they* revise it.

Read the vault profile (`.vault-mind/profile.md`) first if you haven't this session, it
defines this vault's paths and frontmatter schema. If it's missing, run `/vault-mind:init`.
Below, "notes dir", "review-date key", etc. refer to the profile's values.

## Procedure

### 1. Scope the quiz

- **Topic/note given** (`quiz me on X`): gather the relevant notes (match names, tags,
  wikilinks) and any readings the user has marked read on it. These are the source of truth
  for grading.
- **No topic** (`quiz me on what I haven't reviewed`): pick ~5–8 notes by **spacing** ,
  prioritize the oldest review-date (when they last quizzed it), falling back to the
  created-date for notes never reviewed. Favor dense/foundational notes. Spread across areas.
- **Interleave**: when the session spans more than one note/topic, deliberately *mix* them
  rather than clustering all questions on one note (e.g. topic A → topic B → A → C…).
  Interleaving forces discrimination and improves transfer; blocked practice feels easier
  but retains worse. (Within a single-note quiz this doesn't apply.)
- Tell the user what the quiz covers and roughly how many questions, then start.

Useful (run from the vault root; substitute the profile's dir/key names):

```bash
rg -l 'topic' <notes_dir> <resources_dirs>             # files on the topic
rg --files-without-match '^<reviewed_key>:' <notes_dir>/*.md   # notes never reviewed (prime targets)
rg -N '^(<date_key>|<reviewed_key>):' <notes_dir>/*.md         # compare note age vs last review
```

### 2. Ask, one question at a time

- Ask a single question, then **stop and wait** for the answer. Never show the answer with
  the question, never dump a list of questions.
- Mix cognitive levels so it's real recall, not recognition:
  - **Recall**: state what a concept claims.
  - **Application/transfer**: use the idea on a new case.
  - **Connection**: "How does X relate to [[Y]]?" (leverages the Zettelkasten).
  - **Why/derivation**: justify or derive, not just state.
- Prefer open-ended over multiple-choice (open-ended forces generation).
- **Confidence check**: after they answer but before you grade, have them rate how sure they
  are (low / med / high). This calibration is the point of the next step; don't skip it.
- Adapt difficulty to how they're doing.

### 3. Grade against the note + reality

After each answer:
- Compare to the note **and** to correct domain knowledge.
- Be honest and specific: what they nailed, what they missed, what was vague or wrong.
- **Use the confidence rating.** The dangerous quadrant is **high-confidence + wrong**: flag
  those loudly as misconceptions, not slips; they're what active recall exists to catch.
  Low-confidence + right means it's shaky and worth re-quizzing sooner.
- **Classify the gap** so they know the *action*, not just the miss:
  - **faded recall**: they clearly knew it, couldn't retrieve → re-quiz sooner (spacing).
  - **misconception**: confidently wrong / wrong model → `/vault-mind:feynman` it, then revise the note.
  - **never-learned**: the note doesn't cover it / they never had it → learn it from scratch, then write it.
  - **imprecise wording**: right idea, sloppy expression → tighten the note's prose.
- **If the note itself is wrong, outdated, or thin, flag it**: "your note says X but that's
  imprecise because…", so *they* can fix it. Do not fix it for them.
- Keep it tight; then move to the next question.

### 4. Stamp the review (spacing)

For each note actually quizzed, set its review-date key (per the profile) to today's date so
future "what haven't I reviewed" sessions space correctly. This is the only write the skill
makes, a metadata key, never note body. Preserve frontmatter key order; add the key if absent.
(If they're quizzing to *find* weak spots rather than do a real review, ask before stamping.)
If the profile defines no review-date key, say so and offer to add one via `/vault-mind:init`.

### 5. Wrap up

End with a short verbal summary (spoken, not written to the vault): strong areas, weak spots
**grouped by gap type** (above), and which notes to revisit or sharpen. Offer to go again on
the weak ones, interleaving the misconception/faded items first.

If a weak spot is **never-learned** (a real knowledge gap, not faded memory), suggest learning
it from scratch first (an external from-scratch tutor skill, if installed), then writing/revising
the note, then re-quizzing. (This skill only tests what's already in the vault.)
