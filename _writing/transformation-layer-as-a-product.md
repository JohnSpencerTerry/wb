---
layout: post
title: "Treating the transformation layer as a product."
date: 2026-05-01
tags: [Tech]
---

A transformation layer accumulates duplicate logic fast. An analyst needs a number, writes a script, and pushes the result to a table in Postgres. It works. Another analyst needs a slightly different version of the same number and writes another script. An engineer needs it in a pipeline and writes it a third time. Each decision solves an immediate problem.

Three years later, `member_engagement_metrics` exists in four places. They don't agree.

The standard argument is about who should own the SQL. That's not the problem. Analysts already write it, several of them, independently, with no shared definition of what the output is supposed to be. Every time a customer questions a number, tracing it back through three notebooks and a stored procedure to find out which version of the logic produced it costs the team days.

The harder question isn't who writes the SQL. It's whether anyone actually owns it.

## Nobody decided what the transformation layer was for

This stack isn't unusual. Most data teams at a certain age have some version of this: ingestion code in one place, transformation logic scattered across notebooks and scripts and stored procedures, results persisted somewhere shared that everyone reads but nobody fully owns. The specific tools change — pandas or PySpark, Postgres or S3 — but the dysfunction is the same.

The instinct when you see this is to blame the tools. If the team were using the right technology, the thinking goes, this wouldn't have happened. But clean tools produce messy systems just as often as messy tools produce pipelines that run reliably for years. Nobody ever decided what the transformation layer was supposed to be: who it served, who owned it, and how anyone would know if it broke.

That's an architectural decision most new teams skip, because in the early days the cost is invisible.

## When the cost becomes visible

The cost shows up first for whoever trains models on the output. A feature pulled from `fct_member_engagement_metrics` looks reasonable in development, then something downstream doesn't match: an audit report, a number a stakeholder questions. The pipeline itself didn't fail. The raw data is fine. The issue sits in the transformation layer, in logic that exists in three places and has quietly diverged.

The damage stays silent. The pipeline doesn't fail. The model trains. The report generates. Everything looks like it's working right up until someone asks a hard question about a specific number.

That's the worst kind of bug to have in a data system. A pipeline that crashes is easy to fix. A pipeline that produces plausible wrong answers for months is the one that erodes trust in the entire platform.

## What having ownership means

Two positions are both defensible on their own: analysts should write their own SQL, since they understand the business logic best, and uncoordinated SQL ownership is how a metric ends up defined four different ways. Both can be true at once, and the system is still broken. The question of who writes the SQL matters less than the question of whether anyone can stand behind what it produces.

Good SQL isn't the missing piece. What's missing is a single place where the definition lives, a test that catches the logic drifting, and a path back from any number to the code that produced it without days of archaeology.

Having ownership means something specific: there is one definition, it lives in one place, it has tests that enforce what it's supposed to produce, and anyone can trace a number back to the logic that generated it. Those requirements live at the level of discipline, not tooling. It's a decision to treat the transformation layer as something with consumers and contracts, rather than as a collection of scripts that happen to produce the right answer most of the time.

The harder part is cultural. A team can adopt the best tooling available and still end up with four versions of `member_engagement_metrics` if it hasn't agreed on what the transformation layer is supposed to be. Excellent SQL with no shared contract, no tests, and no traceable lineage produces exactly this. Those are solvable problems, but solving them requires agreeing they're problems first.

Every one of those four versions felt like a reasonable decision when someone wrote it. Three years of reasonable individual decisions is exactly what produces this.

## What comes next

Once the problem is agreed on — one definition, one place, testable, traceable — the next question is what that looks like inside a real stack. How do you structure the transformation layer so models build on each other cleanly? Where do the contracts live? How does it fit into an orchestration layer that isn't going anywhere?