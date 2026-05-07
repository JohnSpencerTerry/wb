---
layout: default
title: "dbt models, refs, and sources — a practical intro."
date: 2026-05-07
---

In the [last post](/articles/sql-ownership-problem/), we talked about the ownership problem. The transformation layer at Clarafield had grown into something nobody fully trusted — logic copied between notebooks, SQL duplicated across stored procedures, Devon's engagement model quietly trained on data that had diverged from what production was actually computing. The argument was that treating your transformation layer as a product, with real contracts and clear ownership, was the fix.

This post is about what that looks like in practice. Specifically: the three [dbt](https://www.getdbt.com/product/what-is-dbt) primitives you need to understand to build a structured transformation layer, and how Clarafield's model hierarchy actually maps onto them.

---

## The three things dbt actually gives you

There's a lot of noise around dbt. The pitch can sound like "write SQL, get lineage and docs for free," which is true but undersells the actual value. The real thing dbt gives you is *structure* — a way to build a transformation layer where the dependencies are explicit, the contracts are checkable, and the execution order isn't something you have to track yourself.

Three primitives carry most of that weight: **models**, **sources**, and **refs**.

---

## Models are just SQL files — and that's the point

A dbt model is a `.sql` file that contains a `SELECT` statement. That's it. dbt wraps it in a `CREATE TABLE AS` or `CREATE VIEW AS` depending on your [materialization](https://docs.getdbt.com/docs/build/materializations) config and handles the execution. You write the logic; dbt handles the plumbing.

Here's what `stg_card_transactions` looks like at Clarafield:

```sql
-- models/staging/stg_card_transactions.sql

select
    evt_id                               as event_id,
    acct_ref                             as account_id,
    txn_amt_cents                        as amount_cents,
    txn_amt_cents / 100.0                as amount_dollars,
    txn_ts                               as transacted_at,
    mcc_cd                               as merchant_category_code,
    status_cd                            as status,
    _ingested_at
from {{ source('payments_raw', 'card_events') }}
where status_cd != 'VOID'
```

A few things to notice here. First, the column renames: the raw Kafka event schema uses abbreviated field names that made sense when the pipeline was first built and are now just noise for anyone reading downstream SQL. The staging model is the one place where those names get translated. Every model that builds on top of `stg_card_transactions` uses `amount_cents` and `transacted_at` and doesn't care what the source called them.

Second, the `{{ source(...) }}` — that's the next primitive.

---

## Sources tell dbt where your raw data lives

A source is how dbt refers to data that exists outside the transformation layer — tables your pipelines write directly to S3/Glue, raw Postgres exports, whatever lands in your warehouse before dbt touches it.

At Clarafield, the payments pipeline writes card events into the Glue catalog as `payments_raw.card_events`. dbt doesn't own that table. It just needs to know it exists.

You declare it in a `sources.yml`:

```yaml
# models/staging/sources.yml

version: 2

sources:
  - name: payments_raw
    database: clarafield_lakehouse
    schema: payments
    tables:
      - name: card_events
        description: "Raw card transaction events from Kafka. Updated continuously."
        loaded_at_field: _ingested_at
        freshness:
          warn_after: {count: 1, period: hour}
          error_after: {count: 4, period: hour}
```

Two things the source declaration buys you beyond just telling dbt where the table is. One is the `freshness` check — dbt can test whether the source data is actually fresh before running the models that depend on it. That `loaded_at_field: _ingested_at` tells dbt which column to check. If the ingestion pipeline fails silently and the raw table goes stale, dbt will flag it before your marts compute on bad data.

The second thing is lineage. Because `stg_card_transactions` references `{{ source('payments_raw', 'card_events') }}` instead of hard-coding the table name, dbt knows about the dependency. The lineage graph starts at the raw source and flows through every model that builds on top of it.

---

## Refs are how models depend on each other

Once you have a staging model, you build on it with `{{ ref() }}`. This is the glue of the whole system.

Here's `int_payment_reconciliation`, an intermediate model that joins transaction events against member eligibility to flag mismatches:

```sql
-- models/intermediate/int_payment_reconciliation.sql

with transactions as (
    select * from {{ ref('stg_card_transactions') }}
),

members as (
    select * from {{ ref('stg_member_eligibility') }}
)

select
    t.event_id,
    t.account_id,
    t.amount_dollars,
    t.transacted_at,
    t.merchant_category_code,
    m.member_id,
    m.plan_id,
    case
        when m.member_id is null then true
        else false
    end                                  as is_unmatched
from transactions t
left join members m
    on t.account_id = m.account_id
    and t.transacted_at between m.coverage_start and m.coverage_end
```

Every `{{ ref() }}` call does two things: it resolves to the correct table name in your target schema at runtime, and it registers a dependency edge in dbt's DAG. When you run `dbt run`, dbt builds the graph from these dependencies and figures out the execution order. You don't maintain that order manually. You don't write a topological sort. You just write refs.

The practical consequence: when you run `dbt run --select int_payment_reconciliation+`, dbt knows it also needs to build `stg_card_transactions` and `stg_member_eligibility` first. The `+` is "and everything upstream." You can also run `+int_payment_reconciliation` for "and everything downstream" — useful when you've changed a staging model and want to rebuild all the marts that inherit from it.

---

## The three-layer model hierarchy

At Clarafield, the transformation layer is organized in three layers. This isn't a dbt invention — it's a convention — but dbt's model structure makes it natural to enforce.

**Staging** (`stg_*`): One model per source table. Rename columns, cast types, apply minimal filtering. No joins. No business logic. `stg_card_transactions` doesn't know what a reconciliation mismatch is.

**Intermediate** (`int_*`): Business logic lives here. Joins happen here. `int_payment_reconciliation` is where transaction events meet member eligibility. Intermediate models are not exposed to analysts directly — they're building blocks.

**Marts** (`fct_*`, `dim_*`): Consumer-facing outputs. `fct_card_transactions` is what analysts query for reporting. `fct_member_engagement_metrics` is what Devon uses for modeling. Marts should be stable interfaces — when they change, downstream consumers know about it.

The ownership rule that follows from this: staging models are owned by the data engineering team (they mirror your raw data contracts). Intermediate models are owned by whoever owns the business logic (often a shared responsibility between engineering and analytics). Marts are owned by whoever's accountable for what those numbers mean.

Marcus — Clarafield's analytics lead — initially pushed back on the intermediate layer. His read was that it was an extra abstraction between him and the data. What changed his mind wasn't an argument about dbt; it was realizing that the reconciliation logic in `int_payment_reconciliation` was logic he'd been duplicating in four different analyst queries, and each copy had drifted. The intermediate model is where that logic lives now. His queries got shorter, not longer.

---

## How this solves the actual problem

[Post 1](/articles/sql-ownership-problem/) named three symptoms: logic in multiple places, no way to know which version was right, and no path from a number back to the code that produced it. These three primitives address all three directly.

**One definition, one place.** The reconciliation logic that used to exist in four analyst queries now lives in `int_payment_reconciliation`. Anyone who needs it uses `{{ ref('int_payment_reconciliation') }}`. There is no second copy to drift.

**Testable.** Because models are declared in one place with explicit inputs, you can write tests against them. That's the next post — but the reason testing is even possible at this level of granularity is because the models are discrete, named things with known schemas. You can't test what you can't point at.

**Traceable.** Every `{{ ref() }}` and `{{ source() }}` call is a dependency edge. dbt builds the full lineage graph from those edges. When Devon asks where `is_unmatched` comes from, the answer isn't "ask Priya." It's `dbt docs generate && dbt docs serve` and click through.

Next up: those models need tests. Declaring the structure is step one. Enforcing the contracts — making sure `event_id` is actually unique, `account_id` never nulls, `is_unmatched` only has two valid states — is step two, and it's where dbt testing earns its keep.