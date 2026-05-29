---
layout: default
title: "Modular design decision docs."
date: 2026-05-25
category: Software Engineering
draft: false
---

## Alignment lives at the facet level

Tech leads often need a way to align on a specific decision point, like the canonical source for a record. Two source systems disagree about which one wins when fields conflict, and product, engineering, and analytics each have a stake in the answer. The team needs to land on a rule before anyone builds.

This is facet-level alignment — agreement on one specific choice that downstream work depends on. Tech leads run into a lot of these: this resolution rule, this contract boundary, this approach to backfilling. The project moves or stalls on those decisions.

The instinct when stakeholders are misaligned is to write more — a longer doc, more context, more options, more diagrams. That instinct makes the alignment harder.

## Big documents create feedback fatigue

The default move is to put everything in one place: requirements, architecture, options, the decision. That works as a project artifact, but it's a poor instrument for alignment on any single point.

Reviewers open a long doc, skim, leave a comment on the first thing that catches their eye, and move on. Product cares about the requirements section. Engineering cares about the system diagram. Analytics cares about the output schema. They read different parts and miss the decision the team needs to make. The result is a long comment thread that doesn't converge on the question you wrote the doc to answer.

The decision then gets implicit sign-off. Nobody objected to it, so it's treated as settled. But nobody read it carefully either. Two weeks into implementation, when the work goes in a direction someone didn't expect, the conversation restarts.

## Modular decision docs target one facet

A design decision doc is the alternative. One doc per decision. The whole artifact exists to get the team aligned on one specific choice: the canonical source rule, the contract boundary, the backfill approach.

The small surface area is the point. A reader opens it knowing what's being asked. There's no requirements section to skim, no architecture diagram to comment on instead. Engaging with the doc means engaging with the decision.

These coexist with project docs. A larger initiative still has its PRD or high-level design, and the decision docs sit underneath, one per specific question the team needs to settle. When a project has five real decision points, that's five short docs, each with the right reviewers and the right scope.

Not every decision needs a doc. If a Slack thread or a five-minute chat with the right person resolves it, do that instead. A decision doc earns its existence when the choice is non-obvious, the stakeholders have different priorities, and reversing it later would be expensive. A signal you're over-producing: the doc gets one "looks good" and is signed off in five minutes. That's a decision that could have been a message.

## A worked example: the canonical record decision

Take a typical StartupTechCo scenario. The team operates two upstream systems that both ingest customer records. Call them the operations platform and the legacy CRM. Both write to the lakehouse, and `dim_members` is supposed to expose a single canonical view to the rest of the org. The two systems agree most of the time, but they disagree on roughly 4% of records, usually on status fields, contact info, or the timestamp of the last update.

The data engineer is about to start the work and needs a resolution rule. Product owns the rule logic. The analytics lead consumes `dim_members` downstream and has opinions about which fields he relies on.

Here's what the doc looks like.

### Overview

> Two upstream systems write customer records to staging: `stg_ops_members` and `stg_legacy_crm_members`. On ~4% of records, the two systems disagree on at least one field, most often `status`, `contact_email`, or `last_updated_at`. `dim_members` is the single canonical view downstream, and it currently picks arbitrarily on conflict.
>
> The decision: how should `dim_members` resolve field-level conflicts between the two source systems? Implementation starts next sprint.

### Solution

> Three approaches are on the table.
>
> **Option A: system precedence.** Prefer one source system entirely. `stg_ops_members` wins on any conflict. Simple to implement and explain. Loses information when the legacy CRM has the more current value for a field, which happens for status changes during a known transition window.
>
> **Option B: most-recent write per field.** Resolve each field by `last_updated_at`. Captures the most current value across systems. Requires trustworthy timestamps from both sources. The legacy CRM has known clock skew on overnight batches.
>
> **Option C: per-field rules.** Map each field to a designated source: contact info from `stg_ops_members`, status from `stg_legacy_crm_members`, name fields from whichever is more recent. More accurate but introduces a rules table that product has to own.
>
> Recommendation: Option C. The two systems have known field-level ownership in practice, and a rules table makes that explicit. The tradeoff is that product now owns a small mapping, which they're willing to take on.

### Open questions

> 1. Who owns the rules table going forward? Suggested: product, with a change-review process when a field's source changes.
> 2. What's the default for fields not in the table? Suggested: fall through to Option A (`stg_ops_members` wins).
> 3. How are conflicts logged? Suggested: a `member_conflicts` audit table written by the same dbt model, surfaced in the existing data quality dashboard.

The whole doc fits on a page, and the asks at the bottom name who needs to weigh in on what.

## Narrow the ask

When the team is stuck on something like the canonical record decision, the tempting move is to write more. Add the missing context, add another option, draw the diagram more carefully. That gives the reader more to skim past.

Cut the scope instead. Narrow what you're asking the team to commit to until it fits one decision. The smaller the ask, the easier the answer.

A tech lead's job in these moments is getting the right decision in front of the right people in a shape they can engage with. A modular design decision doc is a cheap way to do that.

