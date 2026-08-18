# Unified Ingestion & Transformation Pipeline — Real-Time Fraud Detection + Batch AML

A conceptual architecture for a scalable data platform that serves two very different financial
workloads on **one shared foundation**: low-latency **payment fraud detection** (streaming) and
nightly **anti-money-laundering (AML) monitoring** (batch) — with the governance, audit and
historical-lookback guarantees a regulated environment needs.

*MSc Data Analytics — Data Intensive Scalable Systems. Architecture design study. Group project (2 members).*

> **Scope:** this is a **design/architecture project**, not an implementation — the deliverable is the
> proposed system design, technology justification, and data-flow, not runnable code.

## The two use cases

- **Real-time payments fraud detection** — an unbounded stream of transaction events that must be
  scored in seconds (block / challenge / allow), where fraud only shows up in recent short-window
  behaviour, so batch-only processing isn't enough.
- **Nightly AML monitoring** — end-of-day batch over a rolling ~90-day lookback, with complex joins
  across ledgers and KYC data to surface layering patterns, producing alerts with an audit trail.

The design point: these have opposite latency needs but overlap heavily in data, so they're built on
one governed platform instead of two separate stacks.

## Architecture

| Layer | Technology | Why |
|---|---|---|
| **Ingestion** | Apache Kafka | Partitioned topics, schema registry, dead-letter queue for invalid records; decouples producers/consumers and scales horizontally |
| **Storage** | Lakehouse — S3 + Parquet + Apache Iceberg | Medallion (Bronze/Silver/Gold); ACID + versioning for reproducible, auditable history |
| **Processing** | Apache Spark | Structured Streaming for fraud scoring **and** nightly batch for AML in one engine (Airflow-orchestrated) |
| **Output/delivery** | PostgreSQL (ACID case DB) + optional OpenSearch + dashboards | Transactional integrity and audit trail for alerts; fast investigator triage |

Key design decisions that make it credible in a regulated setting: idempotent upserts and watermarking
to handle late/out-of-order events and reversals, checkpointing for safe restarts, event-time joins to
**versioned** KYC snapshots so AML results stay reproducible for auditors, and a DLQ so bad records
never break the pipeline.

## Tech stack (conceptual)

Apache Kafka · Apache Spark (Structured Streaming + batch) · Apache Iceberg · Parquet · AWS S3 ·
medallion architecture · Airflow · PostgreSQL · OpenSearch

## Repository structure

```
.
├── report/     # IEEE-format design report (cover sheet & internal identifiers removed)
├── images/     # architecture diagram
└── README.md
```

> The report here has the submission cover sheet and internal identifiers removed. This is a design
> study, so it contains no datasets or customer data.

## Team & contribution

Group project by Chanlin Naicker and Ya Wai Thone (MSc Data Analytics, National College of Ireland).
[LinkedIn](https://www.linkedin.com/in/ya-wai-thone/) · [GitHub](https://github.com/yawaithone)
