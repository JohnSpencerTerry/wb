---
name: proofread
description: Use this skill when the user wants to proofread, review, or polish a draft article on this site. Reads the target article and the relevant plan doc (tech, book review, or life), summarizes the article's argument back as a sense-check, auto-applies low-risk fixes (typos, obvious grammar, punctuation), and proposes voice / clarity / meaning fixes for the user to approve. Preserves voice and intent. Invoked as `/proofread <path-or-slug>`.
---

# Proofread

Reviews a draft article for clarity, voice, grammar, and writing-style violations. Auto-applies low-risk fixes; proposes higher-risk fixes for the user to approve. Works across categories (Software Engineering, Book Reviews, Life) by reading the appropriate plan doc.

The headline check: does the article say what the author meant, and can a reader understand it. Grammar and punctuation are downstream of that.

## Invocation

`/proofread <path-or-slug>` — examples:
- `/proofread each-integration-gets-its-own-role` → resolves to `_articles/each-integration-gets-its-own-role.md`.
- `/proofread _articles/misty-turns-thirty.md` → full path.
- `/proofread` (no arg) → ask the user which file to proofread before proceeding.

## Inputs and references

Before starting, read:

1. The target article file.
2. The article's `category` from frontmatter, which determines which plan doc to load:
   - `Software Engineering` → `_docs/tech_article_plan.md`
   - `Book Reviews` → `_docs/book_review_plan.md`
   - `Life` or anything else → no plan doc; rely on the writing-style memory rules only.
3. The user's writing-style feedback memory (loaded automatically into context via `MEMORY.md`). Key rules to enforce in step 4:
   - No setup-payoff hooks ("the real reason," "but here's the thing," "what I didn't realize was").
   - No narrative scaffolding, scene-setting, or dramatic framing in tech articles.
   - No "X is Y, not Z" / "X is at Y, not Z" contrast pairs (drop the "not Z" half).
   - No triplet negations after a colon ("X does Y: no a, no b, no c").
   - No negation-fragment sentences echoing a positive claim.
   - No unnecessary qualifiers ("actually," "essentially," "basically," "really").
   - No bulleted lists where prose would do.
   - Em-dashes used sparingly.
   - No AI-vocabulary clusters, no "is/are" avoidance, no participial filler, no travel-guide tone, no vague attributions (see category 5 in the writing-style memory and Wikipedia's [signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing) for the full list and examples). The framing John uses: the goal isn't to evade AI detection, it's that these patterns are bad writing regardless of authorship.

## The five steps

Execute in order. Confirm with the user at each gate.

### Step 1 — Resolve the target and read context

1. Resolve the input:
   - Full path (contains `/` or `\`): use as-is.
   - Slug only (no slash, no extension): resolve to `_articles/<slug>.md`.
   - Filename with extension but no path: try `_articles/<filename>`.
2. Read the article file. Identify the `category` from frontmatter.
3. Read the matching plan doc per the mapping above (skip if `Life` or other).
4. Check the `draft:` flag. If `draft: false` (the article is published), ask the user to confirm before proceeding — published articles should not be edited without explicit intent.

### Step 2 — Sense check the argument

Goal: confirm the article's meaning is intelligible before touching anything. If the article can't be summarized cleanly, that's the most important finding.

1. Read the article in full.
2. Summarize back in 2–3 sentences:
   - For tech articles: the opening problem, the main argument, the takeaway.
   - For book reviews: the takeaway and the recommendation.
   - For Life articles: the central scene or claim.
3. Ask the user: "Did I capture what you meant? Anything I missed or misread?"
4. If the user corrects the summary: note each correction as a clarity finding to address in step 4.
5. If the article was hard to summarize: surface that explicitly. It may have a structural problem worth fixing before line-level edits.

Do not move past step 2 until the user confirms the summary (or accepts the noted clarity issues).

### Step 3 — Auto-apply low-risk fixes

Apply directly without asking, then report what changed. Low-risk = no judgment call required and no risk of altering meaning.

In scope for auto-apply:
- Typos and misspellings.
- Obvious grammar: subject-verb agreement, tense consistency, article usage (`a` / `an` / `the`), pronoun reference where unambiguous.
- Punctuation: missing commas, wrong apostrophes (e.g. `it's` vs `its`), sentence-boundary errors, missing terminal periods.
- Trailing whitespace and stray formatting nits.
- Sentence-case consistency in section headings (per the title convention in the plan docs).

NOT in scope for auto-apply (these go to step 4):
- Anything that changes the meaning of a sentence.
- Anything that requires choosing between two valid alternatives.
- Voice or style rewrites.
- Any restructuring.

Report the auto-applied changes as a list (location + before → after). Keep the list short and scannable.

### Step 4 — Propose high-risk fixes

Identify but do NOT apply. Present each as a discrete proposal the user can accept or reject.

**Meaning and clarity:**
- Ambiguous sentences where the referent is unclear.
- Claims that contradict another part of the article.
- Places where the argument doesn't follow from the prior paragraph.
- Sentences the reader has to parse twice.
- Any clarity issue surfaced in step 2.

**Voice and writing style** (from the writing-style memory):
- Setup-payoff hook framings.
- Narrative scaffolding, scene-setting, dramatic framing.
- "X is Y, not Z" contrast pairs.
- Triplet negations after a colon.
- Negation-fragment sentences appended to positive claims.
- Manufactured suspense.
- Unnecessary qualifiers.
- Em-dashes used as a crutch (count them; flag if more than a few in a short article).

**AI vocabulary and structural tics** (writing-style memory category 5, drawn from [WP:AISIGNS](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing)):
- Overused AI vocabulary. Scan for: "delve," "tapestry," "vibrant," "intricate," "pivotal," "underscore," "foster," "garner," "interplay," "landscape," "testament," "showcasing," "boasts," "bolstered," "meticulous," "nestled," "rich heritage," "align with," "enhance," "fostering." One instance is fine; flag clusters or any use in a register where the word doesn't belong.
- "Is/are" avoidance. Flag substitutions like "serves as," "represents," "stands as," "marks," "features," "offers" where plain "is" or "does" would be more direct.
- Participial filler. "-ing" phrases that add no information ("creating a lively community within its borders," "underscoring the importance of...").
- Promotional / travel-guide tone. "Nestled in the heart of...," "rich heritage," "natural beauty," excessive positive framing. Especially in Life-category articles.
- Vague attributions. "Researchers argue," "observers have cited," "many believe" without a named source.

**Structural / formatting checks** (also from WP:AISIGNS):
- Section headings in sentence case, not title case ("Each integration gets its own role." not "Each Integration Gets Its Own Role.").
- Straight quotes (`"` `'`) in markdown source, not curly/smart quotes (`"` `"` `'` `'`).
- No skipped heading levels (`##` → `####` without `###` between).
- Inline-header lists with bolded labels and colons (`• **Label:** description`) are an AI formatting tell — use prose unless the items are genuinely parallel.

**Plan-doc violations** (category-specific):
- *Tech articles:* opens with a definition rather than a concrete problem; uses a non-`StartupTechCo` company name; uses a named character instead of role-based personas; section heading is a topic, not a claim; closing doesn't tie back to opening.
- *Book reviews:* paraphrases the book's table of contents or recaps the plot; opens with synopsis instead of takeaway; missing honest critique; missing recommendation.
- *Life articles:* any fabricated detail (per the personal-articles feedback memory — never invent biographical, scene, or geography details the user didn't supply).

Present each proposal as:
- **Location:** line number or quoted phrase.
- **Issue:** the rule being violated, in one line.
- **Proposed rewrite:** the actual change.

Ask the user to accept or reject each. Apply accepted changes; skip rejected ones.

### Step 5 — Final report

Summarize:
- Auto-applied in step 3: count and short list.
- Proposed in step 4: count, with accepted vs rejected breakdown.
- Confirm `draft:` flag is unchanged.

Tell the user the file is ready for their final review.

## What this skill does not do

- Does not change the article's meaning or voice without user approval.
- Does not flip the `draft: true` flag — that's the user's call.
- Does not edit a published article (`draft: false`) without explicit user confirmation in step 1.
- Does not rewrite sections wholesale — proposals are sentence- or paragraph-level.
- Does not invent claims, examples, or details the article doesn't already contain.
- Does not work on non-article files (code, config, plan docs). Use a different tool for those.
