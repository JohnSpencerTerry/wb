# Tech article plan
## Technical writing — context for all posts

---

## Purpose of this document

This is the canonical reference for the technical writing on this site. It covers only the tech articles. Personal/Life-category articles are handled separately and are out of scope here.

---

## Purpose of the writing

Three things, in priority order:

1. **Demonstrate technical expertise.** The posts are evidence of how the author actually thinks through real engineering problems — opinionated, grounded in production stacks, willing to take a position.
2. **Share thoughts.** A place for ideas that are worth saying out loud — architectural opinions, lessons from things that broke, takes on industry conversations.
3. **Provide a learning resource.** For engineers running into the same problems for the first time. The bar is "would a senior engineer find this useful," not "is this comprehensive."

These goals reinforce each other. The fastest way to demonstrate expertise is to write something genuinely useful. The fastest way to be useful is to share a real opinion grounded in real experience.

---

## About the author

John Terry is a senior software engineer specializing in data platforms. Real-world stack includes Spark on EMR, Airflow, dbt, Iceberg, Kafka, Python, PostgreSQL, AWS, and healthcare data standards (FHIR R4, CCLF, BCDA). Has shipped production pipelines at health tech and fintech companies.

**Voice:** Opinionated, practical, honest about tradeoffs. Shows the real config, names the actual failure mode, explains why the obvious first approach didn't work. Writes like a senior engineer talking to another engineer, not like documentation.

**What this is not:** A survey of the ecosystem. Posts don't compare every tool in a category — they explain a specific decision, why, and what was learned.

---

## Topic areas

The site is organized around four topic areas. Each post sits in one. A topic area is not an arc — posts within an area don't need to build on each other.

### Data engineering
The home base. Transformation layers, orchestration, lakehouse architecture, storage formats, ingestion pipelines, data contracts, testing, observability. This is where the deepest expertise lives.

Examples of post angles: dbt structure and testing; Iceberg internals and operations; Airflow patterns; Spark on EMR; CCLF/BCDA/FHIR ingestion; Kafka → Iceberg streaming.

### AI
Applied AI from a platform engineer's perspective — not model research. The questions that matter here: what does the data platform owe an AI workload, where does reproducibility break, what does an AI-powered feature actually need from the data layer to be trustworthy in production.

Examples of post angles: feature stores and point-in-time correctness; LLM evals as a data problem; MLflow as the data eng/ML handoff layer; building agentic systems on top of structured data.

### Data analysis
The seam between engineering and analytics. SQL patterns, modeling decisions, what makes a metric trustworthy, the difference between a query that runs and a query that's right.

Examples of post angles: the ownership problem in transformation layers; what a mart should actually expose; semantic layer tradeoffs; debugging a number that's "off."

### Product engineering
Engineering decisions that affect what gets shipped. Less about a specific tool, more about how to make good calls under real constraints — staffing, deadlines, legacy systems, blast radius.

Examples of post angles: design docs that actually get read; how to scope a migration; the cost of one more abstraction; what to do when you inherit a system you don't trust.

---

## Naming conventions (generic company / personas)

Use these consistently across all tech posts.

**Company:** `StartupTechCo` — a generic placeholder. Avoid inventing alternate names. Avoid implying a specific industry unless the post requires it (most don't).

**Personas:** When a post benefits from a human voice, use role-based personas instead of named characters.

- **The data engineer** — pragmatic, has been on the team a while, built the old version of whatever is being rebuilt. Speaks from experience, not theory. Pronoun: she/her.
- **The analytics lead** — writes excellent SQL, skeptical of complexity without payoff. The "but why, though?" voice. Pronoun: he/him.
- **The data scientist** — newer to the team, fintech background, runs into problems that turn out to be pipeline problems rather than model problems. Pronoun: he/him.

Use personas only when they add something concrete — a perspective the technical prose alone wouldn't surface. Don't force them in as scenery.

---

## Naming conventions (code examples)

Keep example identifiers consistent across posts so the snippets feel like the same codebase.

**Staging models:** `stg_card_transactions`, `stg_member_eligibility`, `stg_cclf_claims`, `stg_bcda_eob`

**Intermediate models:** `int_payment_reconciliation`, `int_attributed_members`, `int_care_gaps`

**Mart models:** `fct_card_transactions`, `fct_member_quality_measures`, `dim_members`

**Tables / catalogs:** Use generic names like `payments_raw.card_events`. Don't prefix with a company-specific lakehouse name unless the post needs to.

---

## Writing rules

### Lead with the technical problem

Every post opens with a concrete engineering problem — a failure, a constraint, a decision that had to be made. Not a definition. Not a history of the topic.

Good: "The `relationships` test caught a timing issue: transactions were arriving for `account_id` values that hadn't loaded into `dim_members` yet."

Weak: "Data quality is important. In this post, we'll explore how dbt helps teams enforce it."

### Show the real thing

When a post involves configuration, show actual config. When it involves SQL, write the SQL. When it involves a failure mode, name the error and explain what caused it. The value of the post is in the specifics.

### Be opinionated about tradeoffs

Don't hedge everything. The reader should finish each post knowing where the author actually landed, not just what the options were. Acknowledge real limits — "this approach works until X" is more useful than pretending there are no tradeoffs.

### Don't frame posts around tool comparisons

The argument is almost never "tool A vs. tool B." It's "here's the specific problem, here's what worked and why." Lead with the problem, let the tool choice follow.

### Structure

- Open with a concrete problem or moment, not a definition.
- Section headers should be claims, not topics. ("dbt tests are the cheapest way to enforce data contracts" not "dbt testing")
- Sections should build — each one should require what came before. A flat list of topics is a sign the post needs to be reorganized around an argument.
- End with a closing argument that connects back to the opening.

### Length

As short as possible without losing context. Stop when the point is made. Length is earned by argument and example, not by adding sections. If a sentence, paragraph, or section can be cut without losing the argument, cut it.

### Tone — things to avoid

- **Em-dashes used as a stylistic crutch.** Use sparingly; prefer a period or comma when the sentence can take one.
- **"It's not X, it's Y" framings.** The setup-payoff pattern reads as AI-generated. State the point directly.
- **"The real reason is..." / "But here's the thing..."** Trite hooks. Just say the thing.
- **Manufactured suspense.** No "I was about to learn something I'd never forget." Lead with the technical fact.
- **Unnecessary qualifiers.** "Actually," "essentially," "basically" — usually deletable.
- **Bulleted lists where prose would do.** Lists are for parallel items. Don't fragment an argument into bullets.

---

## Published articles

These are locked — future posts should be consistent with what's already on the site.

- **[You Don't Have a SQL Problem. You Have an Ownership Problem.](/articles/sql-ownership-problem/)** — Data analysis / Data engineering. Argues that scattered transformation logic is an ownership problem, not a SQL quality problem. Introduces the `member_engagement_metrics` failure (definition existed in four places, didn't agree). The data engineer and analytics lead's disagreement resolves by recognizing the real question is "does anyone own what the SQL promises."
- **[dbt models, refs, and sources — a practical intro.](/articles/dbt-practical-intro/)** — Data engineering. Walks through the three dbt primitives. Establishes the staging → intermediate → mart hierarchy. Uses `stg_card_transactions` and `int_payment_reconciliation` as the running examples.
- **[Testing your data contracts with dbt: schema tests, custom tests, and CI.](/articles/dbt-testing/)** — Data engineering. Generic tests on `stg_card_transactions`; `relationships` test catching the pipeline ordering bug; `severity: warn` for `amount_cents >= 0` (negative amounts are valid reversals); GitHub Actions CI scoped to the staging layer.

---

## Planned articles

A backlog of post ideas, grouped by topic area. Not a fixed schedule — pick what's most interesting next. Each entry names the technical focus that anchors the post.

### Data engineering

- **Orchestrating dbt runs in Airflow — patterns, pitfalls, and production lessons.** Cosmos integration; the Spark → dbt handoff pattern; passing Airflow context to dbt vars; why DAG structure matters for incremental models.
- **Lineage, documentation, and discoverability: dbt as your data catalog.** `dbt docs generate`; exposures for tracking mart consumers; solving the "where did this feature come from" problem; the limits of dbt docs as a real catalog.
- **Why table formats matter: the problem Iceberg, Delta, and Hudi solve.** The 45-minute query; Hive partitioning limits; why CMS audit time travel is a storage-layer problem, not an application-layer problem.
- **Apache Iceberg internals: snapshots, manifests, and time travel explained.** Metadata hierarchy (table metadata → manifest list → manifest file → data file); snapshot chain; how hidden partitioning works; what `AS OF` actually reads.
- **Running Iceberg on AWS EMR with Spark — a hands-on setup guide.** SparkSession config; Glue catalog integration; common failure modes (catalog permission errors, snapshot isolation edge cases, EMR Serverless cold start behavior with Iceberg).
- **Schema evolution and partition evolution: where Iceberg beats Hive.** CCLF v8 migration story; `ALTER TABLE` without rewrite; partition spec changes without data movement; `MERGE INTO` for row-level updates.
- **FHIR for engineers: what it is, why it matters, and how it maps to real data.** FHIR resource model; ExplanationOfBenefit anatomy; BCDA API flow; written for engineers with fintech background, not healthcare.
- **Kafka to Spark Structured Streaming: building your first real-time pipeline.** Output modes and when each is appropriate; watermark configuration; writing micro-batch output to Iceberg.
- **Terraform for data platform engineers: managing EMR, Glue, and S3 as code.** The undocumented EMR cluster story; module structure for data platform IaC; IAM per workload; CI drift detection.
- **The data contract pattern in practice: what I learned defining CCLF and FHIR schemas.** All four contract components (schema, semantics, SLAs, lineage); the hidden dependency problem; v1 → v2 migration.
- **Rebuilding a CCLF ingestion pipeline on EMR Serverless: what changed, what got better, what broke first.** The old pipeline's latency and observability gaps; why EMR Serverless was the right primitive; the resilience patterns (retries, idempotent writes, checkpointing) that mattered; the EMR-Serverless-specific failure modes that didn't exist on classic EMR.
- **Pre-fetching and enriching payment data with Kafka: a fintech enrichment pattern.** The synchronous-enrichment failure mode that pushed errors into the pipeline; the architectural choice between consumer-side and producer-side enrichment; what observability you only get when enrichment is its own service.
- **Patient medications as a derived data model.** Designing a clinical truth layer downstream teams can trust. NDC code variability, fill-date semantics, the difference between "active medication" and "currently dispensed," and why this is a data engineering decision before it's a clinical one.
- **Quality measures are data engineering, not clinical logic.** HEDIS / STAR measures as testable contracts; how to keep clinical SMEs and engineers in sync; the case for treating each measure as its own mart model with explicit numerator / denominator / exclusion stages.
- **Geospatial data in the warehouse — what to materialize and what to leave to the query engine.** Where Postgis stops being enough; when a search index (ElasticSearch / OpenSearch) is the right answer for read patterns; what a "geospatial mart" actually looks like.

### AI

- **Designing data pipelines for ML: reproducibility, lineage, and feature freshness.** Point-in-time correctness; training/serving symmetry; what "reproducible training run" actually requires from the data platform.
- **MLflow for data engineers: tracking experiments without touching model code.** Pipeline metadata logging (not model metrics); MLflow as the data eng / ML handoff layer; minimum viable integration that doesn't require the ML team to change how they work.
- **The audit trail problem in healthcare AI.** What reproducibility actually requires when a regulator can ask for the inputs to a model decision two years later. Iceberg snapshots as the substrate; model registry pins; deterministic feature snapshots; what a "frozen training input" looks like in practice.
- **Medication adherence prediction is a pipeline problem before it's a modeling problem.** Selection bias enters at ingestion. The fill-date semantics decision changes the label distribution. The data contract for the training set is the model's first hyperparameter.
- **An LLM feature on top of claims data — the data layer, not the prompt.** PHI handling and de-identification at the pipeline boundary; what to log when the output is a generated summary; evals when the ground truth is itself a human judgment call.
- **Feature stores and point-in-time correctness for healthcare workloads.** Why "yesterday's data" can be the right training answer and the wrong inference answer; training/serving skew that comes from clinical event timing, not engineering bugs.

### Product engineering

- **What "enterprise-ready" actually means in healthcare payments.** Audit, traceability, reversibility, SLAs. The features that don't ship until the audit story does.
- **Migrating from on-prem Kubernetes to AWS without a freeze.** Anatomy of a cutover that didn't stop the world. Traffic shifting, dual-write windows, the rollback plan you hope to never use, the observability gap that almost killed it.
- **SOC2 changed how I write integration services.** Identity, secrets, and the "where does this credential live" question. SSO (IdP- and SP-initiated) as the smallest meaningful piece of the picture.
- **Cutting six figures of cloud spend without breaking anything.** The boring wins (rightsizing, retention policies, dev environment lifecycle) beat the clever ones. How to make the case to leadership when the savings are invisible to users.
- **Publishing a Python SDK for an internal API: when the SDK is the product decision.** Versioning, auth, error surface design. Why "we have an OpenAPI spec, isn't that enough" usually isn't.

### Data analysis

- **The quality measure that disagreed with itself.** A debug story about a number that was wrong, traced through staging, intermediate, and mart layers. The fix wasn't in SQL — it was in the contract for the upstream claims dataset.
- **Derived datasets are products, not artifacts.** A clinical truth layer (medications, eligibility, attribution) has consumers, SLAs, and a backlog. Treating it as a product changes how you scope, version, and deprecate it.
- **Diagnosing a Postgres locking issue in a Python/Flask app.** What `pg_stat_activity` actually tells you when queries start piling up; the query pattern that caused it; the fix that wasn't an index.
- **What a healthcare mart should expose — and what it shouldn't.** Member-grain vs. claim-grain marts; the temptation to expose intermediate models; why the answer to "can analytics just query the staging layer" is no, even when it would be faster.

---

## How to use this document

When starting a new tech post:

1. Identify the topic area (data engineering, AI, data analysis, product engineering).
2. Check the published articles for anything that constrains or connects to the new post.
3. Use `StartupTechCo` and the role-based personas — don't invent new company names or named characters.
4. Lead with the technical problem.
5. Apply the writing rules — especially the tone-things-to-avoid list during the final pass.
