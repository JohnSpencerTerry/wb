---
layout: post
title: "Apache Iceberg internals: snapshots, manifests, and time travel."
date: 2026-05-21
tags: [Tech]
draft: false
---

A scheduled merge into `dim_members` overwrote the `member_id` for a few hundred rows. The job had run the same way every day for months. The upstream system added a small batch of records whose source-system email collided with members already in the table, and the merge, keyed on email, picked the new row's id over the existing one. Two distinct people, one identity.

The write itself was well-formed. Every column matched the table schema, every row had the right types, nothing about it tripped a validation. The logic that produced the values was wrong in a way nobody had thought to test for, and the table accepted it.

The bug surfaced hours later, during a routine data quality check that flagged broken referential integrity in a downstream mart. By then the bad snapshot was the current snapshot, and downstream consumers had been reading from it.

The fix took one command: roll back to the previous snapshot. The table was correct again, and the investigation happened later.

On a Hive-style table sitting on top of Parquet in S3, this incident is a simple backup-restore problem. You find the last good copy of the affected partitions, restore them, and assess if downstream consumers have cached or re-aggregated the bad state. Iceberg makes it a metadata pointer change because the previous state of the table is still there, sitting in the snapshot log.

## The table lives in metadata

Look at an Iceberg table on S3 and there are two parallel trees. One holds the data, the other holds the metadata.

```
s3://lakehouse/db/dim_members/
├── data/
│   ├── 00000-0-abc.parquet
│   ├── 00001-0-def.parquet
│   └── 00002-0-ghi.parquet
└── metadata/
    ├── v23.metadata.json
    ├── snap-7283495732-1-xyz.avro
    └── 0a1b2c-m0.avro
```

The Parquet files in `data/` are what you'd expect. They hold rows, and they get written and never edited in place. Everything interesting lives in `metadata/`.

A read of `dim_members` walks [four levels](https://iceberg.apache.org/spec/#table-metadata):

1. **Table metadata file** (`v23.metadata.json`). The entry point. Among other things, it names the current snapshot id.
2. **Snapshot.** Points to one manifest list. Each snapshot is the complete state of the table at one point in time, not a diff from the previous snapshot.
3. **Manifest list** (the `snap-...avro` file). Lists every manifest file in this snapshot, with per-manifest summary stats used for partition pruning.
4. **Manifest files** (`...m0.avro`). Each one tracks a set of data files, with row counts and column-level min/max stats.

<figure class="diagram-figure">
  <svg viewBox="0 0 640 400" role="img" aria-labelledby="awb-tree-title awb-tree-desc" style="width:100%;height:auto;font-family:var(--font-sans);">
    <title id="awb-tree-title">The Iceberg metadata tree for dim_members</title>
    <desc id="awb-tree-desc">Table metadata file points to a snapshot, which points to a manifest list, which points to a manifest file, which tracks three Parquet data files.</desc>

    <line x1="320" y1="62" x2="320" y2="98" stroke="var(--color-hairline)" stroke-width="1.5" />
    <line x1="320" y1="142" x2="320" y2="178" stroke="var(--color-hairline)" stroke-width="1.5" />
    <line x1="320" y1="222" x2="320" y2="258" stroke="var(--color-hairline)" stroke-width="1.5" />
    <line x1="320" y1="302" x2="120" y2="338" stroke="var(--color-hairline)" stroke-width="1.5" />
    <line x1="320" y1="302" x2="320" y2="338" stroke="var(--color-hairline)" stroke-width="1.5" />
    <line x1="320" y1="302" x2="520" y2="338" stroke="var(--color-hairline)" stroke-width="1.5" />

    <rect x="200" y="20" width="240" height="42" rx="3" fill="none" stroke="var(--color-ink-mid)" stroke-width="1.5" />
    <text x="320" y="46" fill="var(--color-ink)" font-size="14" text-anchor="middle">v23.metadata.json</text>

    <rect x="220" y="100" width="200" height="42" rx="3" fill="none" stroke="var(--color-ink-mid)" stroke-width="1.5" />
    <text x="320" y="126" fill="var(--color-ink)" font-size="14" text-anchor="middle">Snapshot</text>

    <rect x="180" y="180" width="280" height="42" rx="3" fill="none" stroke="var(--color-ink-mid)" stroke-width="1.5" />
    <text x="320" y="206" fill="var(--color-ink)" font-size="14" text-anchor="middle">snap-7283495732-1-xyz.avro</text>

    <rect x="220" y="260" width="200" height="42" rx="3" fill="none" stroke="var(--color-ink-mid)" stroke-width="1.5" />
    <text x="320" y="286" fill="var(--color-ink)" font-size="14" text-anchor="middle">0a1b2c-m0.avro</text>

    <rect x="30" y="340" width="180" height="40" rx="3" fill="var(--color-bg)" stroke="var(--color-accent)" stroke-width="1.5" />
    <text x="120" y="365" fill="var(--color-ink)" font-size="12" text-anchor="middle">00000-0-abc.parquet</text>

    <rect x="230" y="340" width="180" height="40" rx="3" fill="var(--color-bg)" stroke="var(--color-accent)" stroke-width="1.5" />
    <text x="320" y="365" fill="var(--color-ink)" font-size="12" text-anchor="middle">00001-0-def.parquet</text>

    <rect x="430" y="340" width="180" height="40" rx="3" fill="var(--color-bg)" stroke="var(--color-accent)" stroke-width="1.5" />
    <text x="520" y="365" fill="var(--color-ink)" font-size="12" text-anchor="middle">00002-0-ghi.parquet</text>
  </svg>
  <figcaption style="font-family:var(--font-sans);font-size:13px;color:var(--color-muted);margin-top:8px;">The four metadata levels from the file listing above, drawn as the tree they actually form. Everything above the data files is metadata; nothing here is a copy of the rows themselves.</figcaption>
</figure>

Then come the Parquet files. The data layer.

The shape of "the table" is the shape of this metadata tree. Adding rows means writing new Parquet files and writing new metadata that points to them. Deleting rows means writing new metadata that doesn't. The Parquet files from older snapshots stay where they are until something explicitly cleans them up. That's why the previous state of `dim_members` was still there to roll back to. Iceberg never overwrote the data files for the bad write; it added new ones and pointed the table at them.

## Snapshots make rollback a pointer change

Every Iceberg table exposes its snapshot chain. In Spark:

```sql
SELECT snapshot_id, parent_id, committed_at, operation
FROM lakehouse.db.dim_members.snapshots
ORDER BY committed_at DESC;
```

That returns the linked list of snapshots in reverse-chronological order. Each row's `parent_id` is the snapshot that came before it. The bad write that started this post was the most recent row. The previous good state was its parent.

Rolling back is a single [procedure call](https://iceberg.apache.org/docs/nightly/spark-procedures/#rollback_to_snapshot):

```sql
CALL lakehouse.system.rollback_to_snapshot('db.dim_members', 7283495732);
```

No data files are read, copied, or rewritten. The table metadata file gets a new version where the current snapshot id points at the parent instead of the bad write. Any read of `dim_members` after that walks the metadata tree from the rolled-back snapshot and gets the previous state.

<figure class="diagram-figure">
  <svg viewBox="0 0 640 240" role="img" aria-labelledby="awb-rollback-title awb-rollback-desc" style="width:100%;height:auto;font-family:var(--font-sans);">
    <title id="awb-rollback-title">Rollback as a pointer change across the snapshot chain</title>
    <desc id="awb-rollback-desc">Before rollback, the current pointer points at the bad write snapshot. After rollback, the current pointer points at its parent instead. The bad write snapshot is still present in the chain either way.</desc>

    <text x="10" y="45" fill="var(--color-muted-strong)" font-size="13">Before</text>
    <line x1="90" y1="70" x2="230" y2="70" stroke="var(--color-hairline)" stroke-width="1.5" marker-end="url(#awb-arrow)" />
    <line x1="270" y1="70" x2="410" y2="70" stroke="var(--color-hairline)" stroke-width="1.5" marker-end="url(#awb-arrow)" />
    <rect x="10" y="50" width="80" height="40" rx="3" fill="none" stroke="var(--color-ink-mid)" stroke-width="1.5" />
    <text x="50" y="75" fill="var(--color-ink)" font-size="13" text-anchor="middle">S1</text>
    <rect x="230" y="50" width="80" height="40" rx="3" fill="none" stroke="var(--color-ink-mid)" stroke-width="1.5" />
    <text x="270" y="75" fill="var(--color-ink)" font-size="13" text-anchor="middle">S2</text>
    <rect x="410" y="50" width="100" height="40" rx="3" fill="var(--color-bg)" stroke="var(--color-accent)" stroke-width="2" />
    <text x="460" y="75" fill="var(--color-ink)" font-size="13" text-anchor="middle">S3 (bad)</text>
    <text x="460" y="35" fill="var(--color-accent)" font-size="12" text-anchor="middle">current</text>

    <text x="10" y="145" fill="var(--color-muted-strong)" font-size="13">After</text>
    <line x1="90" y1="170" x2="230" y2="170" stroke="var(--color-hairline)" stroke-width="1.5" marker-end="url(#awb-arrow)" />
    <line x1="270" y1="170" x2="410" y2="170" stroke="var(--color-hairline)" stroke-width="1.5" marker-end="url(#awb-arrow)" />
    <rect x="10" y="150" width="80" height="40" rx="3" fill="none" stroke="var(--color-ink-mid)" stroke-width="1.5" />
    <text x="50" y="175" fill="var(--color-ink)" font-size="13" text-anchor="middle">S1</text>
    <rect x="230" y="150" width="80" height="40" rx="3" fill="var(--color-bg)" stroke="var(--color-accent)" stroke-width="2" />
    <text x="270" y="175" fill="var(--color-ink)" font-size="13" text-anchor="middle">S2</text>
    <text x="270" y="135" fill="var(--color-accent)" font-size="12" text-anchor="middle">current</text>
    <rect x="410" y="150" width="100" height="40" rx="3" fill="none" stroke="var(--color-muted)" stroke-width="1.5" stroke-dasharray="4 3" />
    <text x="460" y="175" fill="var(--color-muted)" font-size="13" text-anchor="middle">S3 (bad)</text>

    <defs>
      <marker id="awb-arrow" markerWidth="8" markerHeight="8" refX="6" refY="3" orient="auto">
        <path d="M0,0 L6,3 L0,6 Z" fill="var(--color-hairline)" />
      </marker>
    </defs>
  </svg>
  <figcaption style="font-family:var(--font-sans);font-size:13px;color:var(--color-muted);margin-top:8px;">Rollback moves the current pointer from S3 to its parent, S2. S3 is still sitting in the chain afterward — nothing was deleted, which is why the operation is instant.</figcaption>
</figure>

[Time travel](https://iceberg.apache.org/docs/nightly/spark-queries/#time-travel) is the same operation with a different starting snapshot:

```sql
SELECT *
FROM lakehouse.db.dim_members
FOR VERSION AS OF 7283495732;
```

Or by timestamp:

```sql
SELECT *
FROM lakehouse.db.dim_members
FOR TIMESTAMP AS OF '2026-05-20 09:00:00';
```

Both run on a live table. Neither rolls anything back. The query plan picks a snapshot id, walks the manifest list and manifest files attached to that snapshot, and reads the same Parquet files a read would have hit at that point in time.

This is why all three operations are cheap. They do the same thing: choose a snapshot id, walk the manifest list it points to, read the data files those manifests track. Nothing has to be reconstructed from a log of changes.

## Reject the bad write before it lands

The `dim_members` bug had a well-formed schema. Snapshots handle that case. A different class of problem reaches the write boundary with a schema that doesn't match the table: a column type that drifted upstream, a nullable field that's suddenly required, a renamed column that breaks every downstream join. For that, you want the write to fail before it commits.

At work, every batch headed for an Iceberg staging table runs through a PyArrow schema check first. Iceberg's own schema for the table is the source of truth, and PyArrow gives you a clean way to compare an incoming batch's schema to it. Using [`pyiceberg`](https://py.iceberg.apache.org/):

```python
from pyiceberg.catalog import load_catalog
import pyarrow as pa

catalog = load_catalog("lakehouse")
table = catalog.load_table("db.stg_member_events")
expected_schema = table.schema().as_arrow()

def validate_batch(batch: pa.Table) -> None:
    if not batch.schema.equals(expected_schema, check_metadata=False):
        raise SchemaMismatchError(
            f"Incoming batch schema does not match db.stg_member_events.\n"
            f"Expected: {expected_schema}\n"
            f"Got: {batch.schema}"
        )
```

The check runs on the writer side, before `batch` is committed to the table. If upstream changed `event_timestamp` from `timestamp[us]` to `string` because a serializer was swapped out, the writer raises immediately. The Iceberg snapshot chain stays clean: no bad commit to roll back, no downstream consumer to notify.

Two things make this worth doing. First, it puts the failure at the point closest to the cause. A stack trace from the writer names the column and the type mismatch. A stack trace from a downstream query three hours later names some null pointer in an aggregation. Second, the table schema stays canonical. PyArrow doesn't get to decide what `db.stg_member_events` looks like; Iceberg does, and the check verifies the batch agrees.

## Iceberg is a table whose state lives in metadata

For a data engineer new to Iceberg, the most useful first mental model is that the table's state lives in metadata, separate from the data files on S3.

Both guardrails in this post work because of it. Snapshot rollback is cheap because the previous state of the table is a snapshot id in the metadata tree, and walking from there reads the same Parquet files that were already on S3. Schema enforcement at write time is possible because the table schema is itself a metadata fact you can read and compare against.

The `dim_members` bug from the lead was fixed with a one-line procedure call. The bad write was still in the snapshot log, alongside the good one before it; moving the table's pointer back was the entire fix. The schema check that runs in front of `stg_member_events` is the same idea running forward instead of backward. The metadata layer is where the table is, and that's where you fix it from.
