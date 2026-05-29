# Book review plan
## Personal book reviews — context for all reviews

---

## Purpose of this document

The canonical reference for book reviews on this site. Covers voice, structure, and conventions for the Book Reviews category. Reviews share the writing rules of the tech articles; this doc captures what's specific to book reviews.

---

## Purpose of the writing

Short, opinionated reviews of books — mostly engineering, occasionally fiction. Three things, in priority order:

1. **Record what stuck.** The point of the review is the takeaway, not the summary. Future-John should read his own review later and remember why the book mattered.
2. **Tell another reader what to expect.** Honest recommendations, including who the book is and isn't for.
3. **Practice writing about what you read.** Writing the review forces a clearer position than just finishing the book.

---

## Voice

Same plain, declarative voice as the tech articles. Opinionated, honest about what the book got wrong, willing to take a position. Not a book report. No plot recap for fiction. No chapter-by-chapter walk for engineering.

---

## Structure

- Open with the takeaway, not the synopsis. The reader doesn't need to know what the book is about before they know why it mattered.
- One or two main takeaways carry the review. More than that and the review is too long.
- Acknowledge what didn't work. A review that's all positive reads as marketing copy.
- Close with the recommendation. Who should read this. Who should skip it.

No section headers in the body unless the review is unusually long. Most reviews should be a handful of paragraphs of prose.

---

## Length

As short as possible without losing context. The takeaway, the honest critique, the recommendation — that's the whole job. If a sentence or paragraph can be cut, cut it.

---

## Engineering books

The takeaways should be specific. What changed about how you write, design, or argue. Don't paraphrase the book's table of contents. If you've already applied something the book taught, name it. If something is overrated or oversold, say so.

Avoid summary phrasing like "the author argues..." Just state the idea and the author's name if needed.

---

## Fiction books

What stuck — a voice, a character, a moment, a situation. Why you keep thinking about it. Honest recommendation: would you suggest this, and to whom. No plot recap. The reader can look up the synopsis.

---

## Frontmatter

```yaml
---
layout: default
title: "Title of the book."
date: YYYY-MM-DD
category: Book Reviews
author: "Author Name"
year: YYYY
genre: engineering
source: "Library (BPL)"
draft: true
---
```

- `title`: book title in sentence case with a terminal period (matches the article title convention).
- `date`: date of the review, not the book.
- `category`: always `Book Reviews`.
- `author`: full name of the author.
- `year`: original publication year (for older editions, the original year, not the reprint).
- `genre`: `engineering`, `fiction`, or `other`.
- `source`: how the book came to John. Free text with a short qualifier in parens when it adds context. Examples: `"Library (BPL)"`, `"Bought"`, `"Borrowed from Dad"`, `"Gifted (birthday 2025)"`, `"Audiobook (Libby)"`.
- `draft: true`: until the review is ready to publish.

---

## File naming

`_articles/<book-slug>.md`, where `<book-slug>` is kebab-case of the book title (or a shortened form for long titles). Examples: `designing-data-intensive-applications.md`, `the-remains-of-the-day.md`.

---

## Tone — things to avoid

Same rules as the tech articles. See the feedback memory on writing style:

- No setup-payoff hook framings ("the real reason," "but here's the thing").
- No "X is Y, not Z" contrast pairs.
- No triplet negations after a colon ("X does Y: no a, no b, no c").
- No manufactured suspense or scene-setting.
- No unnecessary qualifiers ("actually," "essentially," "basically").
- No bulleted lists where prose would do.
- Em-dashes used sparingly.

---

## Published reviews

(List grows as reviews are published.)
