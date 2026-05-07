---
layout: default
title: "You Don't Have a SQL Problem. You Have an Ownership Problem."
date: 2026-05-01
---

Priya can tell you the history of how Clarafield's transformation layer got the way it is. She helped build part of it during the early Clarafield days when getting customers and continuing to exist was the most pressing matter.

In the early days, the pattern made sense. An analyst needed a number, so they wrote a script and pushed the result to a table in Postgres. It worked. Then another analyst needed a slightly different version of the same number, so they wrote another script. Then an engineer needed it in a pipeline, so they wrote it again. Everyone was just solving their immediate problem.

Three years later, `member_engagement_metrics` exists in four places. They don't agree.

When Marcus, Clarafield's analytics lead, says analysts should own the SQL, he's not wrong exactly. The analysts do own it — several of them, independently, with no shared definition of what the output is supposed to promise. That's the problem Priya is trying to name and can't quite articulate.

She's not trying to start a turf war either. She's been here when a number gets questioned by a customer and the team has to spend several hectic days tracing it back through three notebooks and a stored procedure to figure out which version of the logic actually produced it.

The question was never who should write the SQL. The question was really: Did anyone own what the SQL promised?

## The real problem isn't the technology

Clarafield's stack isn't unusual. Most data teams at a certain age have some version of this: ingestion code in one place, transformation logic scattered across notebooks and scripts and stored procedures, results persisted somewhere shared that everyone reads but nobody fully owns. The specific tools change — pandas or PySpark, Postgres or S3 — but the dysfunction is the same.

The instinct when you see this is to blame the tools. If we were just using the right technology, the thinking goes, this wouldn't have happened. But Priya knows better. She's seen clean tools produce messy systems and messy tools produce pipelines that ran reliably for years. The tools aren't the problem. The problem is that nobody ever decided what the transformation layer was supposed to be — who it served, what it promised, and how you'd know if it broke.

That's an architectural decision most new teams skip because in the early days, the cost is invisible.

## When the cost becomes visible

Devon, Clarafield's new data scientist, joined from a fintech background. He's good at building models. What he kept running into at Clarafield wasn't a modeling problem — it was that he couldn't trust the features he was training on.

He'd pull `fct_member_engagement_metrics` for an engagement model, the model would look reasonable in development, and then something downstream wouldn't match. An audit report. A number a stakeholder questioned. He'd bring it to Priya as a pipeline problem. Priya would dig in and find that the issue wasn't in ingestion — the raw data was fine. It was somewhere in the transformation layer, in logic that existed in three places and had quietly diverged.

The damage was always silent until it wasn't. The pipeline didn't fail. The model trained. The report generated. Everything looked like it was working right up until someone asked a hard question about a specific number.

That's the worst kind of bug to have in a data system. A pipeline that crashes is easy to fix. A pipeline that produces plausible wrong answers for months is the one that erodes trust in the entire platform.

## What owning the promise actually means

The argument between Priya and Marcus ends when they both look at the `member_engagement_metrics` situation. Marcus isn't wrong that analysts should write SQL. Priya isn't wrong that uncoordinated SQL ownership is how you end up here. They're both right, and the system is still broken. That's what finally moves the conversation forward — not one of them convincing the other, but both of them agreeing that the question of who writes the SQL is less important than the question of whether anyone can stand behind what it produces.

Marcus's SQL is good. That was never the issue. The issue was that there was no single place where the definition lived, no test that would catch it if the logic drifted, and no way to trace a number back to the code that produced it without days of archaeology.

Owning the promise means something specific: there is one definition, it lives in one place, it has tests that enforce what it's supposed to produce, and anyone can trace a number back to the logic that generated it. Those are the requirements. Not a specific tool, not a specific team. A discipline — and the decision to treat the transformation layer as something with consumers and contracts, not a collection of scripts that happen to produce the right answer most of the time.

The harder part is cultural. You can adopt the best tooling available and still end up with four versions of `member_engagement_metrics` if the team hasn't agreed on what the transformation layer is supposed to be. Marcus writing excellent SQL in isolation isn't the problem. Marcus writing excellent SQL with no shared contract, no tests, and no traceable lineage is. Those are solvable problems. But solving them requires agreeing they're problems first.

Priya knows this because she built the scripts. She knows exactly how reasonable each individual decision felt at the time, and she knows exactly what three years of reasonable individual decisions produces.

## What comes next

Once you've agreed on the problem — one definition, one place, testable, traceable — the next question is what that actually looks like inside a real stack. How do you structure the transformation layer so models build on each other cleanly? Where do the contracts live? How does it fit into an orchestration layer that isn't going anywhere?