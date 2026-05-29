---
name: book-review
description: Use this skill when the user wants to draft a new book review for the site. Guides them through identifying the book and how they acquired it, a web lookup to confirm metadata and surface context, surfacing concrete takeaways with genre-tailored questions, drafting a short review with draft flag enabled, and a final tone review. Anchors decisions against the book review reference doc.
---

# Book review

Drafts a short book review for the site's Book Reviews category. The author (user) is the source of opinion and reaction — your job is to ask the right questions, capture concrete takeaways, and write tight prose. Do not invent reactions the user didn't describe.

## Inputs and references

Before starting, read:

- `_docs/book_review_plan.md` — voice, structure, frontmatter, things to avoid.
- `_articles/` — filter to `category: Book Reviews` to see existing reviews and avoid duplicating a book.

Reviews live in `_articles/<book-slug>.md` with frontmatter:

```yaml
---
layout: default
title: "Title of the book."
date: YYYY-MM-DD
category: Book Reviews
author: "Author Name"
year: YYYY
genre: engineering
source: "Library (NYPL)"
draft: true
---
```

Set `draft: true` from the moment the file is created. Remove it only when the user says the review is ready to publish.

## The six steps

Execute these in order. Confirm with the user after each step before proceeding.

### Step 1 — Identify the book and how it was acquired

Ask:
- Title, author (if known), genre (engineering, fiction, or other).
- How did the book come to John? Library, bought, borrowed, gifted, audiobook, etc. Where it adds context, name the source (e.g. "NYPL", "Amazon", "from Dad", "Libby"). This becomes the `source` field in frontmatter.

Partial info is fine — step 2 fills in what's missing.

### Step 2 — Web lookup for metadata and context

Goal: confirm bibliographic info, and surface context that informs better questions in step 3.

1. Search the web for the book using title + author (if known). Use WebSearch first; WebFetch a credible source (publisher page, Wikipedia, the author's site) if needed.
2. Extract:
   - Author's full name (use this for the `author` field).
   - Original publication year — NOT the reprint year (use this for the `year` field).
   - Genre / subject area (use this to confirm or adjust the `genre` field).
   - Brief synopsis (1–3 sentences) and main themes or concepts the book is known for. Use these to inform step 3's questions only.
3. If the book isn't findable on the web, surface that to the user and ask them directly for the missing fields.

**Important.** Do NOT paste web content into the review. The web lookup is for confirming metadata and asking better questions. The review is the user's own reaction; web summaries do not belong in it.

Report back to the user with the metadata you collected and ask them to confirm or correct it before moving on.

### Step 3 — Get the reader's reaction

Goal: enough concrete material to write a takeaway-driven review.

Ask open-ended follow-ups, tailored to the genre. Use the context from step 2 to ask targeted questions — if the book is known for arguing X, ask whether that landed or whether John pushed back.

**Engineering books:**
- What's the one or two things you actually took away? Specifics, not paraphrases of the book's contents.
- Anything you've already applied at work? How did it land?
- Anything you disagreed with, or where the book was oversold?
- Who would benefit from reading it? Who already knows this stuff?

**Fiction books:**
- What stuck with you? A voice, a character, a moment, a scene?
- Why are you still thinking about it (or why aren't you)?
- Who would you recommend it to, and who would you steer away?
- Anything that didn't work — pacing, an ending that fell flat, a character who didn't land?

Follow up to pin down specifics. Vague takeaways produce vague reviews.

Do not fabricate reactions. If the user can't name a concrete takeaway or a real reason the book stuck, surface that and offer to skip the review or pivot to a different book.

Output: 1–2 concrete takeaways the review will land on.

### Step 4 — Confirm the takeaways and angle

Goal: alignment before drafting.

Restate in 3–4 sentences:
- The book and the genre.
- The one or two takeaways the review will land on.
- The recommendation (who the book is for, who it isn't).

Confirm with the user. Iterate until they signal the angle is right.

### Step 5 — Draft the review

Goal: a complete short draft in one pass.

1. Pick a kebab-case slug from the book title (shortened if the title is long).
2. Create `_articles/<book-slug>.md` with the frontmatter above (always `draft: true`, today's date). Fill in all metadata fields from steps 1–2.
3. Write the review directly. Structure:
   - Open with the takeaway, not the synopsis.
   - One or two paragraphs developing it with specifics from what the user told you in step 3.
   - Honest critique — what didn't work, what's oversold, what's missing.
   - Close with the recommendation.
4. No section headers in the body unless the review is unusually long. Most reviews are a handful of prose paragraphs.
5. Tell the user the file path. Show the draft and apply any edits before moving to the tone pass.

### Step 6 — Tone pass

Goal: scrub the draft of AI-writing tells before the user reviews for publication.

Read the full review and check for, then fix:

- **Setup-payoff hook framings.** "The real reason...", "But here's the thing...", "What I didn't realize was..." — cut or rewrite.
- **"X is Y, not Z" / "X is at Y, not Z" contrast pairs.** Drop the "not Z" half; let the positive claim stand.
- **Triplet negations after a colon.** "X does Y: no a, no b, no c" — drop the negations; the positive claim already says it.
- **Short negation-fragment sentences echoing the prior sentence.** Drop them; they add weight, not information.
- **Manufactured suspense and scene-setting.** Reviews open with the takeaway, not a scene.
- **Unnecessary qualifiers.** "Actually," "essentially," "basically," "really" — usually deletable.
- **Em-dashes used as a stylistic crutch.** Reviews are short; more than a couple is too many.
- **Plot recap (fiction) or table-of-contents paraphrase (engineering).** Cut. The review is about the reader's reaction.

Report what you found and changed. After the tone pass, tell the user the draft is ready for review. Do NOT remove the `draft: true` flag — that's the user's call.

## What this skill does not do

- Does not publish the review. `draft: true` stays until the user explicitly says to remove it.
- Does not invent the reader's reaction. If the user can't supply a concrete takeaway, the review doesn't get written — offer to skip or pivot.
- Does not summarize the book. Reviews are reactions, not summaries.
