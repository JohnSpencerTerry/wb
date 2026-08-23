---
layout: post
title: "Async refinement replaces the requirements meeting."
date: 2026-07-26
category: Software Engineering
draft: true
---

<!--
OUTLINE

1. The problem — the requirements meeting has a cost
   - A 45-minute refinement meeting where engineers over-litigate a simple requirement,
     nothing gets learned, the requirement was already clear
   - The meeting existed to relitigate a decision that had already been made, not to clarify it
   - Name the actual cost: hours of a team's time, recurring, per requirement

2. The process — async refinement
   - Instead of a live meeting, the requirement gets written up and submitted; the team has a
     window to comment if something needs to change; if nothing's flagged, it's settled
   - This is the practice, independent of any specific tool — it's a different venue for
     surfacing disagreement, not a different amount of scrutiny

3. The implementation — the create-task Skill
   - A Claude Skill that accepts input from wherever a task actually originates: a Slack thread,
     a code-level TODO, a plain string description, an HLD/PRD
   - Drafts a scoped task and finds the right epic by reading the team's epics (descriptive
     names) — assumes and verifies, or asks if genuinely unclear
   - Asks its own clarifying questions before finalizing, so ambiguity gets resolved at draft
     time instead of surfacing live in a meeting
   - Submits to the tracker (ClickUp for this team; same pattern works against Jira or anything
     else with an API)

4. The reward / conclusion
   - The 2-day SLA is what lets async actually close things out — a meeting can't have an SLA,
     it's synchronous by construction
   - Low per-run cost (a fraction of a cent) means the team doesn't ration using it the way
     they'd ration a 45-minute meeting
   - Closing: the skill encodes the process, it doesn't replace it — good process (epic
     convention, clarification step, the SLA) is the precondition, not a byproduct
-->

## The requirements meeting has a cost

I was talking to a product manager friend recently who was venting about a 45-minute refinement meeting his team had just sat through. Afterward I asked him what actually got discussed. Were there real edge cases the engineers had a point about? No. The requirement was already clear. He'd made the call. The meeting existed to relitigate a decision that had already been made, not to clarify anything. Nothing was learned in 45 minutes that wasn't already known going in.

That's not a one-off. Refinement meetings recur, per requirement, across a whole team, and the cost is the same shape every time: a room full of engineers spending real hours re-deriving a conclusion someone already reached, because the meeting is the default venue for surfacing disagreement, whether or not there's anything to disagree about.

## Async refinement

The fix isn't less scrutiny, it's a different venue for it. Instead of putting a requirement in front of the team live and talking it through in real time, write it up and submit it. The team gets a window to comment if something's actually wrong with it. If nobody flags anything in that window, it's settled.

That's async refinement: refinement still happens, disagreement is still welcome, but it happens on the team's own schedule instead of everyone's calendar at once. Nobody has to be in a room, or even online at the same moment, for a requirement to get scrutinized properly.

## The create-task Skill

At work, I built a Claude Skill called `create-task` to run that process. It accepts input from wherever a task actually originates in practice: a Slack thread, a code-level `TODO`, a plain string description, an HLD or PRD. In a real org, tasks don't all start from the same place, and a tool that only accepts one input format misses most of where work actually comes from.

From that input, the skill drafts a scoped task and figures out which epic it belongs to by reading the team's existing epics, which have descriptive names. It assumes the most likely epic and states that assumption for verification, or asks directly if the match is genuinely unclear. Before it finalizes anything, it asks its own clarifying questions, so ambiguity in the input gets resolved right there, by whoever's authoring the task, instead of surfacing live in front of the whole team later. That's the real answer to "was anything learned in that meeting": the decision about what the requirement means gets made once, at draft time, by the person who owns the call.

Once the task is drafted and assigned to the right epic, it's submitted straight to the tracker. We use ClickUp, though the same pattern works against Jira or any other system that exposes an API to create issues and epics.

## The reward

None of this works without a process commitment sitting under it. Once a task is submitted, anyone on the team can comment if they think it needs changes, and the team has adopted a standing 2-day SLA to do so. After that window, the task is considered settled. A meeting can't have an SLA. It's synchronous by construction: it has a start time and an end time, and whatever gets resolved has to get resolved inside that window or wait for the next one. An SLA works the opposite way. It gives disagreement a deadline without requiring everyone to be in the same room, or even online at the same time, to hit it.

The other half of the reward is cost. Running `create-task` costs a small fraction of a cent in token cost, and that changes the calculus of when you're willing to use it. Nobody rations a workflow that costs a fraction of a cent per run the way they'd ration calling a 45-minute meeting with six engineers in it. The cheap path is also the fast one, so there's no tension between using it liberally and using it responsibly.

Skill-based workflows like this one are a genuinely low-cost way to raise a team's productivity and standardize how work gets defined. But `create-task` only works because the process underneath it is sound: the convention of descriptively named epics it can match against, the discipline of asking clarifying questions before submission, and a team that actually honors the 2-day SLA instead of letting tasks sit. Take any of those away and the skill just produces fast, cheap, badly-scoped tasks instead of good ones. The meeting was never the only way to resolve a requirement. It was just the default venue, and it was an expensive one.
