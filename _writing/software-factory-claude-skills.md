---
layout: post
title: "Software factory built from Claude Skills."
date: 2026-08-01
tags: [Tech, AI]
draft: false
---

The "software factory" idea going around the industry right now, [described well here](https://addyosmani.com/blog/software-factories/), is many harnessed loops fed by a queue of work and drained through a review gate into production. A "dark" factory ships code straight through, verified only by machines. A "lit" factory keeps a human in that review gate, upstream of production. At work, we've been piloting a first iteration of the lit version, one that's achievable today, built out of three chained Claude Skills.

## Three skills chain from task to reviewed PR

["Async refinement replaces the requirements meeting."](/writing/async-refinement/) covered `create-task`, the first skill in the chain: it turns a Slack thread, a code TODO, a plain description, or a PRD into a scoped task assigned to the right epic. That's where this factory picks up. Once a task exists, two more skills carry it the rest of the way: `plan-from-task` turns the task into a plan, and `implement-from-plan` turns an approved plan into open pull requests. Together the three cover the loop from "here's a task" to "here's an open, reviewed PR," with a human review gate sitting between planning and implementation.

## What plan-from-task produces

`plan-from-task` takes a task and produces a plan. It considers the system design the change touches, the org's existing conventions, best practices for the kind of change being made, and the acceptance criteria the task needs to satisfy. Its output is a breakdown of exactly which PRs are needed and in which projects, scoped small enough to merge against main quickly rather than landing as one large, hard-to-review change.

A person reads the plan and either approves it or sends it back, before any code exists.

## implement-from-plan turns an approved plan into open PRs

Once a plan is approved, `implement-from-plan` executes it. Subagents open the actual PRs, each isolated in its own worktree so parallel changes don't collide. Each subagent follows the org's code-level conventions and test structure, then runs its own self-review pass, checking its own work for correctness, security, and human readability before a human ever looks at the diff.

A plan doesn't have to go through `implement-from-plan` at all. Handing an approved plan straight to Claude works too, when that's the better fit for the change. `implement-from-plan` is for when the plan is better executed as parallel, isolated PRs than as one continuous session.

## The pilot measures velocity, predictability, and cost

The pilot is just starting at work, so there isn't enough data yet to claim results. What it's measuring is three things: whether this increases velocity, whether outcomes get more predictable, and whether it decreases token cost relative to how the same work gets done today. `create-task` alone has already shown real value on its own, which is a good part of the case for extending the same approach through planning and implementation.

## This isn't end-to-end yet

The factory has no queue. An engineer invokes each skill by prompting for it, rather than tasks flowing through the pipeline automatically. That's a real gap next to something like [Stripe's Minions](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents), which routes over a thousand PRs a week through MCP-connected tooling across 400+ internal systems, pre-warmed devboxes, and per-subdirectory conditional agent rules.

Minions is a real, more mature version of the same idea, built on infrastructure most teams don't have yet. Skills are the part any team can build without it. Three skills, invoked by hand, already standardize how a task turns into a plan and a plan turns into a PR. The queue is the next piece to add once that process is proven out, not a prerequisite for starting.