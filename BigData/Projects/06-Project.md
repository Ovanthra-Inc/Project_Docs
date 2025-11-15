# 🎯 **Project: Session Duration Calculation (Beginner → Pro)**

A perfect micro-project for interns to learn foundational Big Data concepts:
→ timestamp processing
→ grouping events
→ identifying session boundaries
→ calculating engagement metrics
→ writing clean/curated data

This is essential for **student engagement scoring**, **time-tracking**, **learning analytics**, **real-time dashboards**, and **AI personalization**.

---

# ✅ **1. Detailed SRS (Software Requirements Specification)**

## **1.1 Project Overview**

This project builds a pipeline to calculate **session duration per student**, using their event stream (page views, clicks, video interactions).
A “session” is defined as a continuous sequence of events separated by **≤ 30 minutes inactivity**.

## **1.2 Functional Requirements**

### **FR-1: Input Event Collection**

Pipeline must accept events with fields:

* student_id
* timestamp
* event_type
* page_url / screen_name
* metadata (scroll depth, time spent etc.)

### **FR-2: Sorting & Grouping**

Events are sorted by:

* student_id → timestamp ascending

### **FR-3: Sessionization Rules**

A new session is created if:

* `timestamp(current_event) - timestamp(previous_event) > 30 minutes`
* Different device/browser (optional)
* New login (optional future enhancement)

### **FR-4: Session Metrics Calculation**

For each session, calculate:

* session_id (UUID)
* session_start
* session_end
* session_duration (seconds)
* event_count
* pages_visited
* average_event_gap (mean idle time)
* active_time (sum of measured time per event)

### **FR-5: Output Tables**

Produce:

* **sessions table (silver)**
* **session_daily_stats (gold)**

### **FR-6: Storage**

Support:

* Parquet (partitioned by date)
* local, S3, or MinIO

### **FR-7: Idempotency**

Re-running pipeline must not duplicate sessions.

### **FR-8: Monitoring**

* Validate missing timestamps
* Validate event order
* Log session boundary anomalies

---

# ✅ **2. Dataset Design**

## **2.1 Input Dataset (Raw Events)**

File: `data/raw/events/date=2025-11-16/*.json`

Example record:

```json
{
  "event_id": "e123",
  "student_id": "stu_1004",
  "timestamp": "2025-11-16T10:15:22Z",
  "event_type": "page_view",
  "page_url": "/chapter/5",
  "device": "android",
  "metadata": {
    "scroll_depth": 78
  }
}
```

## **2.2 Cleaned Event Schema (Silver)**

Columns:

```
event_id
student_hash
event_type
ts
date
course_id
chapter_id
device
metadata
```

## **2.3 Session Table (Silver)**

```
session_id (string, UUID)
student_hash (string)
session_start (timestamp)
session_end (timestamp)
session_duration_sec (int)
event_count (int)
pages_visited (int)
avg_event_gap_sec (float)
device (string)
date (partition)
```

## **2.4 Gold Table: session_daily_stats**

```
date
student_hash
total_sessions
total_time_spent_sec
avg_session_duration
longest_session_sec
shortest_session_sec
```

---

# ✨ **3. Architecture Diagram (Text-based)**

```
                       [Raw Events from LMS Frontend]
                                     |
                                     v
                            Raw Zone (JSON/Parquet)
                                     |
                               Cleaning Script
                                     |
                                     v
                         Clean Events (Silver Zone)
                                     |
                              Sessionization Job
                                     |
                                     v
                          Sessions Table (Silver)
                                     |
                           Daily Aggregation Job
                                     |
                                     v
                  Gold Zone → session_daily_stats (analytics)
                                     |
           ┌─────────────────────────┴──────────────────────────┐
           v                                                    v
   BI Dashboards                                        ML Personalization
```

---

# ✅ **4. Step-by-Step Tasks for Intern**

## **Phase 1 — Beginner (3–4 days)**

### **Task 1: Understand sample events**

* Review provided sample dataset
* Understand timestamp formats (UTC, ISO8601)

### **Task 2: Clean & Normalize Events**

* Convert timestamp → Python datetime
* Remove null timestamps
* Hash student_id
* Sort events

### **Task 3: Implement Session Boundary Logic**

Pseudo-code:

```
session_id++
if event_gap > 30 minutes:
     start new session
else:
     same session
```

### **Task 4: Output Result to Parquet**

* Write sessions to `data/silver/sessions/date=YYYY-MM-DD/`

---

## **Phase 2 — Intermediate (1–2 weeks)**

### **Task 5: Add Aggregations**

* total sessions/day
* total time spent/day
* average session duration

### **Task 6: Build Airflow DAG**

Steps:

* ingest_raw_events
* clean_events
* compute_sessions
* compute_daily_session_stats

### **Task 7: Add Data Quality Checks**

* ensure session_duration > 0
* ensure session_end >= session_start
* ensure no overlapping sessions

---

## **Phase 3 — Pro (1–2 weeks)**

### **Task 8: Real-Time Stream Sessionization (PySpark)**

* use Structured Streaming
* maintain session state
* write micro-batches

### **Task 9: Device-Based Session Rules**

New session if:

* device changed
* browser changed
* network changed

### **Task 10: AI Enhancement**

Predict:

* likely dropout session
* low engagement session

---

# ✅ **5. GitHub Folder Structure**

```
session-duration-pipeline/
│
├── data/
│   ├── raw/events/
│   └── silver/sessions/
│   └── gold/session_daily_stats/
│
├── src/
│   ├── cleaning/
│   │   └── clean_events.py
│   ├── sessionization/
│   │   └── compute_sessions.py
│   ├── aggregation/
│   │   └── daily_session_stats.py
│   ├── airflow/
│   │   └── session_duration_dag.py
│   ├── utils/
│   │   ├── hashing.py
│   │   └── parsers.py
│   └── tests/
│       └── test_session_logic.py
│
├── docs/
│   ├── SRS.md
│   ├── architecture.md
│   └── data_dictionary.md
│
├── notebooks/
│   └── session_duration_eda.ipynb
│
└── README.md
```

---

# 🎓 **6. Beginner → Pro Curriculum (Sessionization-Focused)**

## **Week 1 — Basics**

* JSON, Parquet, CSV
* Timestamps, timezone handling
* Python Pandas basics
* Sorting and grouping data
* 30-minute inactivity rule

## **Week 2 — Data Engineering Concepts**

* ETL (Bronze → Silver → Gold)
* Partitioning
* Session identifiers
* Aggregations
* Parquet optimization

## **Week 3 — Orchestration**

* Airflow DAGs
* Scheduling jobs
* Retry, backfill, SLA

## **Week 4 — Advanced**

* Streaming sessionization (PySpark/Kafka)
* Stateful processing
* Real-time dashboards

## **Week 5 — AI+Engagement Insights**

* Predict long sessions
* Detect anomaly sessions
* Engagement scoring