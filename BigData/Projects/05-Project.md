# 📘 Project: ETL Pipeline for Student Behavior (Beginner → Pro)

A production-ready, end-to-end project plan to teach interns how to build reliable ETL pipelines that convert raw LMS events into analytics-ready tables and ML-ready features. This is the backbone for personalization, churn prediction, mastery scoring, dashboards, and alerts in your AI-first K-12 LMS.

---

# ✅ 1. Detailed SRS (Software Requirements Specification)

## 1.1 Project Overview

Build an automated ETL pipeline that ingests student behavior events (clickstream, video, quiz, assignments), cleans/enriches them, computes sessionization & derived metrics, stores results in a Data Lake and Data Warehouse, and exposes tables/APIs for analytics and ML.

## 1.2 Functional Requirements

* **FR-1 Ingestion:** Accept batched and streaming inputs from:

  * Frontend APIs (POST `/v1/track`)
  * Kafka topics (clickstream, quiz, video)
  * Periodic batch files (mobile analytics SDKs)
* **FR-2 Raw Storage:** Persist raw events in immutable raw zone (S3/MinIO) organized by date/topic.
* **FR-3 Validation & Schema:** Validate events against JSON Schema; reject/log malformed events.
* **FR-4 Cleaning & Enrichment:** Normalize fields, anonymize PII, enrich with geo/ip/device info, map course/chapter metadata.
* **FR-5 Sessionization:** Group events into sessions (session_id) using inactivity threshold (e.g., 30 min).
* **FR-6 Aggregate & Compute Derived Metrics:** session_duration, page_dwell, completion_flag, scroll_depth_avg, attempts_count, time_per_question, video_watch_pct.
* **FR-7 Storage (Curated):** Save cleaned & aggregated tables in parquet (bronze→silver→gold) and push curated tables to Data Warehouse.
* **FR-8 Scheduling & Orchestration:** Use Airflow/Dagster to orchestrate hourly/daily pipelines + backfills.
* **FR-9 Monitoring & Alerting:** Track job success/failure, data quality checks, SLA alerts on missing partitions.
* **FR-10 Access:** SQL-ready tables for analysts, and REST endpoints for dashboards/ML services.
* **FR-11 Rollback & Idempotency:** ETL jobs must be idempotent and support reprocessing/backfill.

## 1.3 Non-Functional Requirements

* **Scalability:** Support 5–10M events/day; horizontally scalable.
* **Latency:** Batch pipelines available within 15–60 minutes; streaming near real-time (<1–5 min) for critical metrics.
* **Reliability:** 99.9% pipeline availability; durable storage with replication.
* **Security & Compliance:** Pseudonymize student_id, encrypt data at rest/transit; role-based access.
* **Cost-efficiency:** Partitioning and compaction to reduce storage/query cost.
* **Observability:** Metrics, logs, lineage (who/what/when), and schema evolution support.

## 1.4 Stakeholders

Product, Data Engineers, Data Analysts, ML Engineers, Teachers (consumers), School Admins.

---

# ✅ 2. Dataset & Table Design

Notes: design follows Bronze → Silver → Gold pattern.

## Raw zone (Bronze) — `events/raw/<topic>/date=YYYY-MM-DD/`

Store raw JSON events as-is (gzip/snappy).

Fields (typical event):

```
{
  "event_id": "uuid",
  "student_id": "uuid",
  "event_type": "page_view|quiz_attempt|video_play|submit_assignment",
  "timestamp": "ISO8601",
  "page_url": "...",
  "course_id": "...",
  "chapter_id": "...",
  "question_id": "...",
  "metadata": { ... }  // scroll_depth, duration, selected_option...
}
```

## Cleaned events table (Silver) — `events.cleaned` (parquet / partition by date)

Columns:

* event_id (string)
* student_hash (string) — hashed/pseudonymized
* event_type (string)
* ts (timestamp)
* date (date) — partition
* course_id (string)
* chapter_id (string)
* session_id (string)
* device_type (string)
* ip_geo (string)
* metadata (map/json)

## Session table (Silver) — `sessions`

* session_id
* student_hash
* session_start
* session_end
* session_duration_seconds
* events_count
* device_type
* is_mobile

## Aggregates / Gold tables (analytics-ready)

### `student_daily_activity`

* date, student_hash, total_sessions, total_time_spent_seconds, pages_viewed, quizzes_attempted, videos_watched_seconds, assignments_submitted

### `chapter_engagement`

* date, course_id, chapter_id, total_time_spent, avg_time_per_visit, unique_students, visits_count, avg_scroll_depth, dropoff_rate

### `question_attempts_summary`

* date, quiz_id, question_id, attempts_count, correct_count, avg_time_spent, difficulty_estimate

### `feature_store.student_features` (ML-ready)

* student_hash, as_of_date, rolling_7d_time_spent, rolling_7d_sessions, avg_quiz_accuracy_30d, last_login_days_ago, engagement_trend

---

# ✅ 3. Architecture Diagram (textual)

```
[Frontend / Mobile SDKs]  ->  [API Gateway] -> [Kafka / Kinesis topics]
                                 |
                                 v
                           [Ingestion Service]
                                 |
                                 v
                             Raw Zone (S3/MinIO)  (Bronze)
                                 |
                           Validation & Schema Registry
                                 |
                                 v
                       Cleaning & Enrichment (Spark/Pandas)  <-- course metadata DB
                                 |
                                 v
                         Sessionization (Spark Structured Streaming or Batch)
                                 |
                                 v
                         Silver Zone (Parquet: cleaned events, sessions)
                                 |
                                 v
                           Aggregations (Spark / DBT)
                                 |
                                 v
                         Gold Zone (Data Warehouse: BigQuery / Snowflake)
                                 |
            ┌────────────────────┴──────────────────────┐
            v                                           v
     BI / Dashboards (Metabase / Superset)        ML / Feature Store
            |                                           |
        Alerts / Notifications                 Model Serving (FastAPI)
```

---

# ✅ 4. Step-by-Step Tasks for the Intern

Break into phases with concrete deliverables.

## Phase 0 — Prep (1–2 days)

* Read SRS.md and walkthrough event sample JSONs.
* Install local tools: Docker, MinIO, Kafka (docker-compose), Airflow (local), Python 3.10+, PySpark (optional).

## Phase 1 — Beginner (3–5 days)

1. **Sample Data**: Create a small synthetic dataset of student events (clicks, video pings, quiz attempts).
2. **Raw Storage**: Write script to save sample events to `data/raw/...` as JSON.
3. **Cleaning Script**: Implement `clean_events.py` using Pandas to:

   * Parse timestamps
   * Normalize event types
   * Hash student_id
   * Fill missing fields
   * Save parquet partitioned by date
4. **Basic Queries**: Write SQL on cleaned parquet to compute DAU, average session length (simple session_id from input).
5. **README**: Document commands and sample outputs.

## Phase 2 — Intermediate (1–2 weeks)

6. **Sessionization**: Implement sessionizer that groups events by student_hash and inactivity threshold (30 min). Output sessions parquet.
7. **ETL Orchestration**: Create Airflow DAG:

   * Task 1: Ingest raw -> store
   * Task 2: Validate & clean
   * Task 3: Sessionize
   * Task 4: Aggregate daily metrics
8. **Enrichment**: Add geo lookup stub (free IP database) and device classification.
9. **Data Warehouse Load**: Create SQL/DDL for `student_daily_activity` and load sample data into local DuckDB or BigQuery sandbox.

## Phase 3 — Advanced (2–3 weeks)

10. **Streaming Consumer**: Build Kafka consumer (PySpark Structured Streaming or Faust) that writes events to raw zone and triggers micro-batches for near-real-time metrics.
11. **DBT Models**: Add DBT models for gold tables; implement tests for nulls, uniqueness.
12. **Feature Store**: Implement simple feature extraction job producing `student_features` for last 7/30 days.
13. **Monitoring & Data Quality**: Integrate Great Expectations checks and Airflow sensors; add simple Grafana dashboard for job metrics.

## Phase 4 — Pro (ongoing)

14. **Backfill & Idempotency**: Implement reprocessing logic and S3 partition management.
15. **Cost & Performance Tuning**: Partitioning strategy, parquet compaction, file sizing.
16. **Security & Governance**: Implement RBAC on Data Warehouse, data lineage, and PII policies.
17. **Deploy**: Prepare k8s deployment manifests or Terraform for infra.
18. **Handoff**: Create runbook, run monthly maintenance, and present.

---

# ✅ 5. GitHub Folder Structure (practical, copyable)

```
etl-student-behavior/
│
├── data/                              # sample/local datasets
│   ├── raw/
│   └── processed/
│
├── docker/                            # docker-compose for dev infra
│   └── docker-compose.yml
│
├── src/
│   ├── ingestion/
│   │   ├── api_producer.py            # simple POST -> kafka producer
│   │   └── kafka_producer_example.js
│   │
│   ├── validation/
│   │   └── schemas/                   # json schemas for events
│   │
│   ├── etl/
│   │   ├── clean_events.py            # pandas / pyspark cleaning
│   │   ├── sessionize.py
│   │   ├── enrich.py                  # geo/device enrich
│   │   └── aggregate_daily.py
│   │
│   ├── streaming/
│   │   └── structured_streaming_job.py
│   │
│   ├── dbt/                           # if using dbt, or separate folder
│   │   └── models/
│   │
│   ├── warehouse/
│   │   └── ddl/                       # SQL DDL for tables
│   │
│   └── tests/
│       └── test_cleaning.py
│
├── airflow/
│   └── dags/
│       └── etl_student_behavior_dag.py
│
├── infra/
│   ├── k8s/                           # optional Helm charts / manifests
│   └── terraform/
│
├── docs/
│   ├── SRS.md
│   ├── data_dictionary.md
│   └── runbook.md
│
├── notebooks/
│   └── eda_student_behavior.ipynb
│
├── requirements.txt
└── README.md
```

---

# ✅ 6. Beginner → Pro Curriculum (ETL-focused)

## Module A — Foundations (1 week)

* Linux & Docker basics
* Python & Pandas
* CSV/JSON/parquet fundamentals
* Basic SQL & DuckDB

## Module B — Batch ETL (2 weeks)

* Writing ETL with Pandas / PySpark
* Partitioning strategies
* Parquet & file sizing
* Airflow fundamentals & DAGs
* Unit tests for ETL

## Module C — Streaming & Real-time (2 weeks)

* Kafka basics: topics, producers, consumers
* PySpark Structured Streaming or Faust
* Exactly-once semantics, idempotency
* Micro-batch vs stream processing

## Module D — Orchestration & Quality (2 weeks)

* Airflow advanced: XComs, SLA, sensors
* DBT modeling & tests
* Great Expectations for data quality
* Monitoring: Prometheus + Grafana + Alerting

## Module E — ML & Production (2–3 weeks)

* Feature engineering & incremental features
* Building `student_features` for ML
* Serving features and models (FastAPI)
* Cost optimization & security best practices

---

# ✅ Example Quick Start (commands interns can run locally)

1. Clone repo & start dev infra:

```bash
git clone <repo>
cd etl-student-behavior
docker-compose -f docker/docker-compose.yml up -d
```

2. Produce sample events:

```bash
python src/ingestion/api_producer.py --sample 100
```

3. Run cleaning job locally:

```bash
python src/etl/clean_events.py --input data/raw/2025-11-15 --output data/processed/2025-11-15
```

4. Start Airflow (dev) and trigger DAG:

```bash
# (outlined in README)
```

---

# ✅ Deliverables for the Intern (milestones)

* M1: Synthetic events + raw storage (day 3)
* M2: Cleaning script + parquet outputs (day 7)
* M3: Sessionization + basic aggregates (day 14)
* M4: Airflow DAG automating steps (day 21)
* M5: Streaming consumer writing to raw zone (day 30)
* M6: Feature store outputs + docs (day 45)

---