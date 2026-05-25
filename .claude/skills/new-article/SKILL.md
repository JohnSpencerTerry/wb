---
name: new-article
description: Use this skill when the user wants to draft a new tech article for the site. Guides the author through topic selection, outline planning (with draft flag enabled), section-by-section writing with user choices at each step, and a final tone review. Anchors decisions against the tech series bible.
---

# New article

Drafts a new tech article for this site through a structured five-step collaboration. The author (user) is the source of judgment and experience — your job is to ask the right questions, capture decisions, and write the prose. Do not invent technical experiences the user didn't describe.

## Inputs and references

Before starting, read:

- `_docs/tech_article_plan.md` — voice, topic areas, naming conventions (StartupTechCo, role-based personas), writing rules, and the tone-avoid list.
- `_articles/` — for the existing tone, structure, and to avoid topic/example collisions with what's already published.

Articles live in `_articles/<slug>.md` with frontmatter:

```yaml
---
layout: default
title: "Title in sentence case with terminal period."
date: YYYY-MM-DD
category: Software Engineering
draft: true
---
```

Set `draft: true` from the moment the file is created. Remove it only when the user says the article is ready to publish.

## The five steps

Execute these in order. Do not skip ahead. After each step, confirm with the user before proceeding to the next.

### Step 1 — Decide on a topic and gather experience

Goal: arrive at a concrete topic the user has real experience with. A vague topic produces a vague article.

1. Ask the user what they want to write about. If they only have a topic area (e.g. "something about AI"), check the **Planned articles** section of `_docs/tech_article_plan.md` and offer 2–3 entries from that list as candidates. If nothing in the planned list fits, propose new options aligned to the topic areas.
2. Once the topic is named, ask about their specific experience: what have they actually done, what broke, what surprised them, what would they tell a colleague who was about to do the same thing. Open-ended questions, not multiple choice.
3. Ask clarifying questions as needed to nail down: the concrete problem the post will lead with, the specific stack/context, the audience (what does the reader already know), and the one thing the reader should walk away believing.
4. Do not fabricate experiences, stack details, or war stories. If the user can't give you a concrete moment to anchor the post, surface that and offer to either narrow the topic or pivot to one they have more to say about.

Output of step 1: a one-paragraph summary of the topic, the lead, the audience, and the takeaway. Confirm with the user before moving to step 2.

### Step 2 — Plan the outline and write it to a draft file

Goal: a section-by-section outline written into the article file with `draft: true`.

1. Pick a slug from the title (kebab-case, no stop words). Confirm with the user if it's not obvious.
2. Create `_articles/<slug>.md` with the frontmatter above (always `draft: true`, today's date as the file is being authored).
3. Write the outline as a structured list inside the file, but commented out or in a clearly-marked "Outline" section at the top of the body. Each section should have:
   - A claim-style header (not a topic label — per series bible)
   - 2–4 bullets capturing the concrete points / examples / code the section will contain, drawn from the user's stated experience
4. Outline should lead with the technical problem (per series bible). End with a closing argument that connects back to the opening.

Output of step 2: the file exists with frontmatter and outline. Tell the user the file path so they can open it.

### Step 3 — Review the outline

Goal: alignment on the structure before writing prose.

1. Ask the user to review the outline directly in the file.
2. Ask explicitly: are the sections in the right order, is anything missing, is anything that's in the outline actually thin (not enough real experience to back it up).
3. Apply their feedback to the outline in the file. If sections get cut, cut them. If they get added, ask follow-up questions about the new section's content before adding it.
4. Iterate until the user signals the outline is good.

Do not start writing prose until the user approves the outline.

### Step 4 — Write the article section by section

Goal: a complete draft, written one section at a time with the user's input at each step.

For each section, in order:

1. Re-read the outline bullet for the section.
2. Before writing, ask the user 1–3 focused choices about what to include. Examples:
   - "For this section we have two examples to choose from — the X case or the Y case. Which one carries the argument better?"
   - "Do you want a code block here? If so, what's the actual snippet — paste it or describe it and I'll draft."
   - "Should this section name the failure mode explicitly or keep it abstract?"
   Use the AskUserQuestion tool when the choices are well-formed; ask in free text when the question is genuinely open.
3. Write the section into the file. Stay within the section's scope — don't drift into territory that belongs to a later section.
4. Show the user what you wrote (or tell them which section to look at in the file) and ask for feedback before moving to the next section.
5. Apply edits before moving on. Do not accumulate a backlog of "I'll fix that later" feedback.

Repeat until all sections are drafted.

### Step 5 — Tone review

Goal: scrub the draft of AI-writing tells before the user reviews for publication.

Read the full article and check for, then fix:

- **Em-dashes used as a stylistic crutch.** Count them. If there are more than ~3 in a 1,500-word post, prune. Prefer periods or commas where the sentence can take one. Em-dashes for genuine parenthetical asides are fine — wholesale replacement is not the goal.
- **"It's not X, it's Y" framings.** Search the draft for "not...but", "isn't...it's", "the real X is". Rewrite to state the point directly.
- **Trite hooks.** "The real reason...", "But here's the thing...", "Here's what I learned...", "Spoiler:". Cut or rewrite.
- **Manufactured suspense.** No "I was about to learn something..." setups. Lead with the technical fact.
- **Unnecessary qualifiers.** "Actually", "essentially", "basically", "really" — usually deletable.
- **Bulleted lists where prose would do.** Lists are for parallel items. If a list is fragmenting an argument, convert to prose.
- **Generic openings.** Confirm the article still opens with a concrete problem, not a definition.
- **Naming consistency.** Company is `StartupTechCo`. Personas are role-based (the data engineer / the analytics lead / the data scientist). No invented company names or named characters.

Report what you found and changed. If anything needed your judgment (e.g. an em-dash that's load-bearing), flag it explicitly so the user can override.

After the tone pass, tell the user the draft is ready for their review. Do NOT remove the `draft: true` flag — that's the user's call when they're ready to publish.

## What this skill does not do

- Does not publish the article. The `draft: true` flag stays until the user explicitly says to remove it.
- Does not invent the user's technical experience. If the user can't supply a concrete moment, the post doesn't get written — pivot the topic instead.
- Does not skip steps. Outline review (step 3) is the most commonly skipped step and the one most often regretted.
