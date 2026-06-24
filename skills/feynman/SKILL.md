---
name: feynman
description: Use when the user wants to explain a concept back in their own words to test their understanding (the Feynman technique) — e.g. "let me explain backprop to you", "feynman me on the Koopman operator", "check my understanding of this concept". They explain; you find the gaps, errors, and hand-waving versus their note and the literature, and push them to sharpen it. Never writes notes.
---

# Feynman (explain-it-back)

The user teaches a concept back to you in their own words; you act as the skeptical student
who exposes where their understanding is incomplete or wrong. The learning happens when *they*
articulate and then repair the gaps — so your job is to find the gaps, not to fill them.

**Core rule: never write note content.** You do not rewrite their note or hand them a clean
explanation to copy. You diagnose; they revise.

Read the vault profile (`.vault-mind/profile.md`) first if you haven't this session (paths +
frontmatter schema). If it's missing, run `/vault-mind:init`.

## Procedure

### 1. Pick the concept and load ground truth

- Identify the note/concept (from their prompt). Read their note in the notes dir and any
  source readings it cites, so you know both *what they wrote* and *what's correct*.
- If they haven't started explaining, prompt them: "Explain <concept> to me from scratch, like
  I know nothing. Go." Then wait.

### 2. Listen, then probe — don't lecture

As they explain (and across follow-ups):
- **Find the holes**: unstated assumptions, skipped steps, vague hand-waving
  ("it just learns the features" — *how?*), wrong causality, conflated ideas.
- **Probe with questions, not answers**: "Why does that step work?", "What happens in the
  edge case where…?", "You said X causes Y — what's the mechanism?", "How is this different
  from [[related concept]]?"
- Make them go one level deeper than they want to. The Feynman test is whether they can explain
  the *why*, not recite the *what*.
- Steelman a wrong-but-common alternative and see if they can refute it.
- **Watch confidence vs correctness.** Note where they assert something *fluently and surely*
  that's actually wrong or hand-waved — confident fluency is where misconceptions hide. Where
  they flag their own uncertainty, that's honest and lower-risk; press the parts they *didn't*
  flag but should have.

### 3. Diagnose against their note and the literature

- Point out specifically where their explanation (and, if relevant, **their existing note**)
  is wrong, imprecise, or missing something — citing the correct understanding so they know
  the target. Naming the gap and the correct fact is fine; that's diagnosis. **Writing the
  paragraph that goes in their note is not** — that's theirs to write.
- **Classify each gap** so the fix is obvious (same taxonomy as `/vault-mind:quiz`):
  - **misconception** — confidently wrong model → the priority; rework the idea, then the note.
  - **never-learned** — they never had it / the note doesn't cover it → learn it, then write it.
  - **imprecise wording** — right idea, sloppy expression → tighten the prose.
  - **faded recall** — they know it but couldn't surface it → `/vault-mind:quiz` it later, no rewrite needed.
- If their note is solid and the gap was just in the verbal explanation, say so.

### 4. Hand it back

End by listing the 1–3 weak spots they should shore up, **tagged by gap type** (above), and
suggest they update *their own* note to close them. Offer to re-run once they've revised.
Create/edit no files.

If a weak spot is **never-learned** (a real conceptual gap, not imprecise wording), suggest
learning it from scratch first (an external from-scratch tutor skill, if installed) before
revising the note. (This skill pressure-tests what they already understand.)
