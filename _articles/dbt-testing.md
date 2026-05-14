---
layout: default
title: "Testing your data contracts with dbt: schema tests, custom tests, and CI."
date: 2026-05-14
category: Software Engineering
---

In the [last post](/articles/dbt-models-refs-sources/), we built out the model structure: staging, intermediate, marts. This post is about testing it — asserting that what those models produce is actually what you expect.

dbt tests are post-execution data quality assertions — they execute against whatever data exists in your target schema (typically a test dataset) and fail if the output violates a condition. You're not aiming for coverage of every column in every model. The goal is to assert the properties that, if violated, would silently corrupt everything downstream.

---

## Generic tests

dbt ships with four generic tests: `unique`, `not_null`, `accepted_values`, and `relationships`. They're declared in a `schema.yml` alongside your model.

```yaml
# models/staging/schema.yml

version: 2

models:
  - name: stg_card_transactions
    description: "Staged card transaction events from Kafka. One row per transaction."
    columns:
      - name: event_id
        description: "Unique identifier for the transaction event."
        tests:
          - unique
          - not_null

      - name: account_id
        description: "Foreign key to dim_members."
        tests:
          - not_null
          - relationships:
              to: ref('dim_members')
              field: account_id

      - name: status
        description: "Transaction status."
        tests:
          - not_null
          - accepted_values:
              values: ['APPROVED', 'DECLINED', 'REVERSED']

      - name: transacted_at
        tests:
          - not_null

      - name: amount_cents
        description: "Transaction amount in cents."
        tests:
          - not_null
```

A null `event_id` means `fct_card_transactions` has null primary keys. A `status` outside the accepted set means mart logic with a `CASE WHEN status = 'APPROVED'` is silently producing wrong numbers for some rows.

The `relationships` test is worth highlighting. At Clarafield, it caught a timing issue: transactions were arriving for `account_id` values that hadn't loaded into `dim_members` yet because two pipelines had no guaranteed ordering in Airflow. The fix was in the DAG, not the dbt model — but the test is what surfaced it.

---

## Custom tests

When the four generic tests aren't enough, you write a singular test: a `.sql` file in the `tests/` directory that returns rows when the assertion fails, zero rows when it passes.

```sql
-- tests/assert_transaction_dates_are_valid.sql

select
    event_id,
    transacted_at
from {{ ref('stg_card_transactions') }}
where
    transacted_at > current_date
    or transacted_at < dateadd(year, -5, current_date)
```

Prefix custom tests with `assert_` to distinguish them from model outputs. dbt discovers any `.sql` file in `tests/` automatically.

Three cases where custom tests are necessary:

**Multi-column uniqueness.** `unique` applies to a single column. If your natural key is `(event_id, line_number)`, `unique` on either column alone will pass and mean nothing.

**Cross-model reconciliation.** `relationships` checks referential integrity. For looser checks — "total `amount_cents` in `stg_card_transactions` should be within 1% of `fct_card_transactions`" — you need a custom assertion.

**Domain-specific validation.** Things the generic tests can't know: dates outside a plausible range, status codes that exist in the raw event but aren't valid for your current schema version.

---

## Severity

Not every failure should stop the pipeline. `severity: warn` runs the test and logs the result without failing `dbt test`.

```yaml
      - name: amount_cents
        tests:
          - not_null
          - dbt_utils.expression_is_true:
              expression: ">= 0"
              severity: warn
```

Negative amounts can be valid — they represent reversals. Worth surfacing, not worth halting a daily mart refresh over. The calibration matters: if everything is an error, teams start ignoring failures. Null primary keys are always errors. Domain anomalies worth investigating are warnings.

---

## CI

Running `dbt test` locally is useful. Running it automatically on every pull request is what makes tests into an enforced contract.

```yaml
# .github/workflows/dbt_ci.yml

name: dbt CI

on:
  pull_request:
    branches: [main]

jobs:
  dbt-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Install dependencies
        run: pip install dbt-spark

      - name: dbt compile
        run: dbt compile --profiles-dir ./profiles

      - name: dbt test (staging layer only)
        run: dbt test --select staging --profiles-dir ./profiles
```

Two things to get right. First, run against a CI environment with representative data — `dbt test` against an empty schema passes every test. Second, `--select staging` scopes tests to the layer where raw data contracts are enforced. If `stg_card_transactions` passes, you've verified ingestion output before any downstream model runs.

---

Next up: these models need to run on a schedule, in the right order, in production. That's [orchestration](/articles/dbt-airflow-cosmos/) — how Airflow and dbt fit together, what Cosmos buys you, and how to handle the Spark → dbt handoff in the same DAG.