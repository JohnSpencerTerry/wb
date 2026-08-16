---
layout: post
title: "Sharing Claude Code skills across projects."
date: 2026-08-16
category: Software Engineering
draft: true
---

<!--
OUTLINE

## Copy-pasting a skill into a second repo is the same DRY problem as copy-pasting code
- The concrete moment: wanting a standard skill (e.g. one that plans a codebase layout/architecture from a PRD or design doc) available in every new repo, not just the one it was first written in — personal projects tend to be POC-type work, but the problem isn't specific to POCs
- Framed as a scale-of-consumers problem, not a single incident: a few dozen engineers at work vs. a handful of personal repos, same underlying issue
- Set up the two halves of the article: what a fuller solution looks like at work, and what's still an open question for individual/POC use

## At work, skills are distributed like packages, not files
- Org-wide and team-specific plugin marketplaces, GitHub-synced
- Namespaced invocation: `org-wide:skill-name`, `my-team:skill-name` — skills resolve by marketplace, not just by name
- Admin controls that come with that infrastructure: installed-by-default, available-for-install, required, per-group overrides
- This is built for an org with many engineers and many repos; the infrastructure cost is justified by the number of consumers
- Link: [Manage plugins for your organization](https://support.claude.com/en/articles/13837433-manage-plugins-for-your-organization)

## For personal or POC repos, that infrastructure is overkill
- No admin, no org, often just one person working across several small repos
- Anthropic's current recommended path for individuals: host the skill folder on GitHub, clone or reference it from each repo — no built-in cross-repo sync or namespacing at this scale
- What the lighter approach looks like concretely: a shared skills repo, cloned or referenced from each project, no admin layer or namespacing needed
- skill-creator as the tool for authoring/reviewing the skill itself, independent of how it eventually gets distributed
- Link: [The Complete Guide to Building Skills for Claude](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf) (distribution and sharing chapter, skill-creator section)

## The marketplace pattern is Claude-specific, not agent-agnostic
- SKILL.md as a format is portable (published as an open standard, works across Claude.ai/Claude Code/API) — but the distribution mechanism (org/team marketplaces, `marketplace:plugin` namespacing, GitHub sync, install controls) is Claude Code/Claude.ai infrastructure
- Building workflow around that mechanism is a real tradeoff: it makes switching to GPT or another model's agent tooling harder, because the skill content moves but the distribution layer doesn't
- Worth naming explicitly before adopting the pattern, not discovering it later

## Closing: right-size the mechanism to the number of consumers
- Org marketplace for dozens of engineers, plain GitHub hosting for a handful of personal repos — same underlying DRY problem, different justified infrastructure
- Connects back to the opening: the goal isn't to copy the org's marketplace pattern into a personal setup, it's to size the sharing mechanism to how many places are actually consuming the skill
-->

## Copy-pasting a skill into a second repo is the same DRY problem as copy-pasting code

I have a Claude Code skill that reads a planning doc and proposes a codebase layout and architecture from it. The first time I wrote it, it lived in the repo I was working in at the time. Later, when I started a new project and wanted the same skill, the easiest option was to copy the `SKILL.md` file over.

That's the same problem as copy-pasting a function into a second repo instead of publishing it as a package. It works, but the skill is logic that will drift the first time either one gets edited, and there's no mechanism forcing them to stay in sync.

At work, this problem multiplies across a few dozen engineers and skills need to encapsulate a lot of institutional conventions. In my own projects, most of which are proof-of-concept work I start and abandon quickly, it's a handful of repos and one person. Same underlying issue, different amount of infrastructure worth building to solve it. My team at work has already built a real answer for the org-scale version. For my own repos, a much lighter approach is the right size.

## At work, skills are distributed like packages, not files

At work, skills don't live in individual repos at all. They live in plugin marketplaces: an org-wide one for skills every engineer should have, and team-specific ones for skills that only make sense within a smaller group. Both sync from a central GitHub repo that engineers, not some third-party admin, own and contribute to. A skill change goes through the same PR review as any other change, which matters, because a bad skill instruction spreads to everyone who has it installed the moment the review approves it. Merge a version bump to the default branch and the update lands automatically.

Skills don't have to start at the org level either. A skill can start in a team's own marketplace, and if it turns out to be useful beyond that team, it gets promoted to the org-wide one. The marketplace boundary tracks how broadly useful the skill has proven to be, not just where it happened to be written.

Because a skill can come from more than one marketplace, invocation is namespaced by where it came from: `org-wide:skill-name`, `my-team:skill-name`. Two marketplaces can each ship a skill with the same short name without colliding, because the name Claude resolves is scoped to the marketplace.

The distribution model goes further than just getting the file onto someone's machine. Admins set a distribution preference per plugin: installed by default for everyone, available for self-service install, hidden from the catalog entirely, or required, meaning it can't be uninstalled. Enterprise admins can override the org-wide setting for specific groups, and when someone belongs to more than one group, the most permissive setting wins.

Across a large engineering team, someone needs to know whether a skill update reached every engineer, and someone needs the authority to keep a required skill from being turned off. The marketplace infrastructure answers both without anyone manually checking in on each repo.

## For personal repos, that infrastructure is overkill

None of that applies to my own projects. There's no admin, no group to override a setting for, and no question of whether an update reached everyone, because there's only one person to reach.

What I do is keep my standard skills in one repo on disk and link `.claude/skills` in each project back to it. On Windows that's a directory junction (`mklink /J`) rather than a true symlink, since Windows needs admin rights or Developer Mode enabled to create real symlinks and a junction doesn't. Either way, edit the skill once and every linked project picks up the change immediately. There's no marketplace, no namespacing, no distribution preference to set, because none of those problems exist at this scale. A plain `git clone` of the shared skills repo into each project would work just as well, and is closer to what Anthropic currently recommends for individuals: host the skill folder on GitHub and reference it from wherever you need it.

Authoring the skill in the first place is the same job regardless of which distribution model it ends up in. I use Anthropic's `skill-creator` skill to build and review new skills before I decide where they live. It asks for the concrete use case, generates the `SKILL.md` frontmatter and instructions, and flags the usual problems (a description too vague to trigger reliably, a missing trigger phrase) before I've spent time on a skill that never fires. That step is identical whether the skill is about to go into an org-wide marketplace or a linked folder on my laptop.

## The marketplace pattern is Claude-specific, not agent-agnostic

The skill format itself is portable. Anthropic has published Agent Skills as an open standard, and a `SKILL.md` file works the same way across Claude.ai, Claude Code, and the API without modification. That's the content.

The distribution layer is a different matter. Org and team plugin marketplaces, GitHub sync, `marketplace:plugin` namespacing, the install and override controls, all of that is Claude Code and Claude.ai infrastructure. None of it has an equivalent in another model's agent tooling. Building the sharing mechanism around it is a real cost worth knowing about upfront: if the team ever needs to switch models or run a second agent alongside Claude, the skill content travels, but the marketplace it lives in doesn't. It's not a reason to avoid the pattern, but it's worth naming before investing in it, not discovering after.

## Right-size the mechanism to the number of consumers

An org-wide marketplace with PR-reviewed contributions and per-group install controls is the right amount of infrastructure for a large engineering team. A single shared repo, linked into each project, is the right amount of infrastructure for one person sharing skills across a handful of personal projects. Copying the same `SKILL.md` around by hand is the wrong amount of infrastructure for either.

The mechanism should match the number of places consuming the skill, not the sophistication of the skill itself. My PRD-to-architecture skill isn't more sophisticated than the ones sitting in my company's org-wide marketplace. It just has fewer consumers, which is exactly why it doesn't need a marketplace of its own.
