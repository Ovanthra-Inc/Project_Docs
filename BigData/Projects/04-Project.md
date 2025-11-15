
# 🚀 **Project: Real-Time Clickstream Ingestion (Kafka Mini Project)**

A beginner–friendly but industry-grade project to teach your interns **Kafka, real-time ingestion, event streaming, and basic analytics** for LMS user behavior.

This project fits perfectly into your AI-first LMS because **all student actions → become live events → feed analytics + personalization**.

---

# ✅ **1. Detailed SRS (Software Requirement Specification)**

## **1.1 Project Overview**

We track **every student click & activity** on the LMS platform in real time and ingest it into a Kafka cluster, store it in a Data Lake, and prepare it for analytics.

Clickstream includes:

* Page visits
* Button clicks
* Scroll events
* Search queries
* Video play/pause
* Quiz interactions
* Time spent ping events

This project lays the foundation for:

* Real-time dashboards
* Engagement analytics
* Recommendations
* Anomaly detection
* Future ML models

---

# **1.2 Functional Requirements**

### **FR-1: Collect Clickstream Events**

Events include:

* student_id
* event_name
* page_url
* chapter_id
* timestamp
* session_id
* device_type
* metadata (JSON)

### **FR-2: Frontend → Kafka Producer**

LMS frontend sends events to:

```
POST /v1/track
```

Node.js backend pushes events → Kafka topic:

```
topic: lms_clickstream
```

### **FR-3: Kafka Consumer**

A Python consumer listens in real time and pushes:

* raw events → Data Lake
* processed events → Data Warehouse

### **FR-4: Data Storage**

Use:

* **Raw Zone** → JSON
* **Processed Zone** → Parquet
* **Warehouse Schema** → Analytics tables

### **FR-5: Real-Time Dashboard (Optional Mini Feature)**

Display:

* live user count
* live page activity
* top pages
* real-time student actions

### **FR-6: Monitoring & Retry**

* Failed events logged
* Retry mechanism
* Kafka lag monitoring

---

# **1.3 Non-Functional Requirements**

* **Scalability:** 50K events/minute
* **Latency:** < 100ms ingestion
* **Fault tolerance:** Kafka replication
* **Durability:** Events stored in Data Lake
* **Security:** Token-based API + masked student IDs
* **Compression:** Snappy/Gzip for Data Lake files

---

# **1.4 Stakeholders**

* Data Engineers
* Backend Engineers
* Data Analysts
* LMS Team
* ML Engineers

---

# 🧱 **2. Dataset Design**

Below are the tables after event processing.

---

## **Table 1 — clickstream_raw (JSON in Data Lake)**

| Field       | Type     |
| ----------- | -------- |
| event_id    | string   |
| student_id  | string   |
| event_name  | string   |
| page_url    | string   |
| chapter_id  | string   |
| session_id  | string   |
| device_type | string   |
| browser     | string   |
| timestamp   | datetime |
| metadata    | JSON     |

Example metadata:

```
{ "scroll_depth": 60, "duration": 120, "query": "polynomial" }
```

---

## **Table 2 — clickstream_processed (Parquet)**

| Field        | Description            |
| ------------ | ---------------------- |
| date         | Partition              |
| student_id   | Hashed/Pseudonymized   |
| session_id   | Session                |
| event_name   | click, view, scroll    |
| page_url     | URL                    |
| chapter_id   | LMS chapter            |
| duration     | time spent if provided |
| scroll_depth | %                      |
| timestamp    | event time             |

---

## **Table 3 — clickstream_analytics**

| Metric               | Meaning               |
| -------------------- | --------------------- |
| active_users         | AMU, DAU              |
| avg_session_duration | mean session time     |
| top_pages            | most visited          |
| scroll_behavior      | engagement metric     |
| click_frequency      | clicks/min user       |
| bounce_rate          | single-event sessions |

---

---

# 🏗️ **3. Architecture Diagram**

```
                     ┌────────────────────────┐
                     │ LMS Frontend (React/JS)│
                     │  → Sends click events  │
                     └─────────────┬──────────┘
                                   │ POST /track
                                   ▼
                     ┌────────────────────────┐
                     │   Node.js API Server   │
                     │Kafka Producer (events) │
                     └─────────────┬──────────┘
                                   │
                           Kafka Broker Cluster
                        ┌──────────┴───────────┐
                        │   Topic: clickstream │
                        └──────────┬───────────┘
                                   │
                    ┌──────────────▼──────────────┐
                    │ Kafka Consumer (Python/PySpark) │
                    │  → Raw Zone (JSON)             │
                    │  → Processed Zone (Parquet)    │
                    └──────────────┬──────────────┘
                                   │
                    ┌──────────────▼──────────────┐
                    │      Data Lake (S3/MinIO)     │
                    └──────────────┬──────────────┘
                                   │
                    ┌──────────────▼──────────────┐
                    │  Data Warehouse (BQ/Redshift) │
                    └──────────────┬──────────────┘
                                   │
                    ┌──────────────▼──────────────┐
                    │ Real-Time Dashboard (Superset) │
                    └──────────────────────────────┘
```

---

# 📝 **4. Step-by-Step Tasks for Intern**

---

# **Phase 1 — Beginner**

### ✔ Task 1: Learn JSON clickstream data

➜ Create sample event JSON
➜ Understand fields & metadata

### ✔ Task 2: Set up Kafka locally (Docker)

Containers:

* kafka
* zookeeper
* kafka-ui

### ✔ Task 3: Create a Kafka topic

```
lms_clickstream
```

### ✔ Task 4: Build Kafka Producer

Small Node.js script to send a sample event every 2 seconds.

### ✔ Task 5: Build Kafka Consumer

Python consumer using:

```
confluent-kafka
```

---

# **Phase 2 — Intermediate**

### ✔ Task 6: Store raw events → Data Lake

folder:

```
/data/raw/clickstream/YYYY/MM/DD/
```

### ✔ Task 7: Convert raw json → processed parquet

using Pandas or PySpark

### ✔ Task 8: Build partition strategy

Partition by:

```
date
event_name
```

### ✔ Task 9: Create Data Warehouse tables

Using BigQuery / Snowflake / Redshift

---

# **Phase 3 — Advanced**

### ✔ Task 10: Real-time consumer with PySpark

Process events at scale.

### ✔ Task 11: Clickstream analytics aggregator

Compute:

* session duration
* scroll depth distribution
* most visited pages

### ✔ Task 12: Build real-time dashboard

Use:

* Apache Superset
* Metabase
* Grafana
* PowerBI

### ✔ Task 13: Create real-time API

```
GET /analytics/live-users
GET /analytics/top-pages
```

---

# **Phase 4 — Pro**

### ✔ Task 14: Add event schema validation

Using:

* JSON Schema
* Kafka Schema Registry

### ✔ Task 15: Event enrichment pipeline

Add:

* geo-location
* device fingerprint
* session reconstruction

### ✔ Task 16: Build anomaly detection

Detect:

* unusual traffic
* bot-like behavior
* sudden spikes

### ✔ Task 17: Build ML-ready feature dataset

* click frequency
* scroll depth
* learning patterns

### ✔ Task 18: Add monitoring

* Kafka lag
* Broken events
* Data Lake ingestion failures

---

# 📁 **5. GitHub Folder Structure**

```
real-time-clickstream-ingestion/
│
├── docker/
│   ├── docker-compose.yml
│
├── frontend/
│   └── click-producer.js
│
├── backend/
│   ├── api-server.js
│   ├── kafka-producer.js
│
├── consumer/
│   ├── consumer.py
│   ├── process_events.py
│
├── data/
│   ├── raw/
│   └── processed/
│
├── pyspark/
│   └── streaming_job.py
│
├── warehouse/
│   └── schema/
│       ├── clickstream_processed.sql
│       ├── clickstream_analytics.sql
│
├── dashboard/
│   └── superset/
│
├── airflow/
│   └── dags/
│       └── clickstream_etl_dag.py
│
└── docs/
    └── SRS.md
```

---

# 🎓 **6. Beginner → Pro Curriculum (Intern Path)**

---

## **Module 1 — Beginner**

* Kafka basics
* Producer/Consumer concepts
* JSON event schema
* Basic Python

---

## **Module 2 — Intermediate**

* Kafka + Node.js producer
* Kafka + Python consumer
* Data Lake partitioning
* Parquet files
* ETL pipelines

---

## **Module 3 — Advanced**

* PySpark streaming
* Real-time processing
* Metrics aggregation
* Building dashboards

---

## **Module 4 — Pro**

* Schema Registry
* Event validation
* Sessionization
* ML-ready pipelines
* Monitoring & observability
