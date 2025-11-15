
# **📘 Project: Time Spent Per Chapter Analysis (Beginner → Pro)**

A structured, real-world project for analyzing how much time students spend on each chapter/module inside your LMS.

---

# ✅ **1. Detailed SRS (Software Requirement Specification)**

## **1.1 Project Overview**

The goal is to track and analyze **time spent by students on each chapter** inside your LMS.
This helps measure:

* Content engagement
* Drop-off chapters
* Difficult chapters (more time spent)
* Improving course design
* Personalized learning recommendations

---

## **1.2 Functional Requirements**

### **FR-1: Track Page/Chapter View Events**

System must capture:

* student_id
* chapter_id
* course_id
* chapter_start_time
* chapter_end_time
* time_spent_seconds
* device_type
* session_id

### **FR-2: Data Ingestion**

Events must be collected through:

* LMS frontend → API
* Event stream (Kafka)
* Mobile app analytics

### **FR-3: Data Storage**

* Raw events stored in Data Lake
* Clean & processed metrics stored in Data Warehouse

### **FR-4: ETL Processing**

System must calculate:

* Time spent per chapter
* Time spent per course
* Average time per visit
* Completion percentage
* Drop-off chapter index

### **FR-5: Analytics**

Dashboards must show:

* Per chapter time spent
* Time distribution across users
* Average vs median time
* Chapters with highest drop rate
* Comparison across courses

### **FR-6: Alerts**

System must generate alerts for:

* Chapters with very low engagement
* Chapters where time spent spikes (possible complex content)
* Students who spend very little time (possible skimming)

---

## **1.3 Non-Functional Requirements**

### **NFR-1: Scalability**

Should handle **5M chapter-view events/day**.

### **NFR-2: Performance**

Chapter analytics dashboards must load under **3 seconds**.

### **NFR-3: Security**

Encrypted student IDs, API-level authentication.

### **NFR-4: Reliability**

Ingestion reliability: **99.9% uptime**.

### **NFR-5: Data Accuracy**

Time tracking error < **1 second**.

---

## **1.4 Stakeholders**

* Data Engineer
* Data Analyst
* Learning Experience Designer
* Course Admin
* Product Manager

---

# ✅ **2. Dataset Design**

## **Table 1: chapter_view_events (raw)**

| Field              | Type        | Description             |
| ------------------ | ----------- | ----------------------- |
| event_id           | UUID        | Unique event            |
| student_id         | string      | User ID                 |
| course_id          | string      | Course reference        |
| chapter_id         | string      | Chapter/module ID       |
| start_time         | datetime    | When chapter was opened |
| end_time           | datetime    | When chapter was closed |
| time_spent_seconds | int         | End - Start             |
| session_id         | string      | Session tracking        |
| device_type        | string      | mobile/desktop          |
| browser            | string      | optional                |
| scroll_depth       | int (0-100) | Engagement indicator    |

---

## **Table 2: chapter_engagement_summary (processed)**

| Field            | Description                            |
| ---------------- | -------------------------------------- |
| student_id       | User                                   |
| course_id        | Course                                 |
| chapter_id       | Chapter                                |
| total_time_spent | Total seconds                          |
| avg_time_spent   | Avg per visit                          |
| visits_count     | Number of times chapter visited        |
| last_visited_at  | Recent time                            |
| completion_rate  | (scroll_depth or last section reached) |
| drop_off         | yes/no                                 |

---

## **Table 3: chapter_analytics (aggregated)**

| Field             | Description                     |
| ----------------- | ------------------------------- |
| chapter_id        | Unique chapter                  |
| avg_time_spent    | Average time spent by all users |
| median_time_spent | Robust metric                   |
| total_visits      | Number of visits                |
| unique_students   | Total users                     |
| drop_off_rate     | % drop before 50% scroll        |
| difficulty_score  | Time spent + drop-offs weighted |

---

# ✅ **3. Architecture Diagram**

```
              ┌──────────────────────────┐
              │       LMS Frontend       │
              │  (Chapter View Events)   │
              └─────────────┬────────────┘
                            │
                            ▼
                  ┌─────────────────┐
                  │  Kafka Stream   │
                  └───────┬─────────┘
                          │
       ┌──────────────────┴──────────────────┐
       │           ETL/ELT Pipeline           │
       │ (Airflow + Spark/Pandas Processing)  │
       └──────────────────┬──────────────────┘
                          │
              ┌───────────┴────────────┐
              │          Data Lake       │
              │      (S3 / MinIO)        │
              └───────────┬────────────┘
                          │
                ┌─────────┴───────────┐
                │   Data Warehouse     │
                │ (BigQuery / Redshift)│
                └─────────┬───────────┘
                          │
   ┌──────────────────────┴───────────────────────┐
   │                 BI & Analytics                │
   │ (PowerBI / Tableau / Looker / Metabase)      │
   └──────────────────────────────────────────────┘
```

---

# ✅ **4. Step-by-Step Tasks for Intern**

## **Phase 1 — Beginner**

1. Understand dataset fields (chapter events)
2. Clean raw data using Python Pandas
3. Calculate per-chapter time spent
4. Chart: **Avg time spent per chapter**
5. Create SQL queries:

   * Chapters with highest time spent
   * Chapters with lowest time spent
   * Top engaged students
   * Drop-off analysis

---

## **Phase 2 — Intermediate**

6. Build Airflow DAG for daily ETL
7. Create S3/MinIO bucket for raw data
8. Partition events by `course_id/date`
9. Create DBT models to compute aggregates
10. Build dashboards:

    * Time per chapter
    * Heatmap: time spent per hour
    * Time spent distribution (box plot)

---

## **Phase 3 — Advanced**

11. Use PySpark for large dataset calculations
12. Real-time chapter engagement analytics with Kafka
13. Identify difficult chapters using ML
14. Build API endpoint: `/v1/analytics/chapter-insights`
15. Add pipeline monitoring

---

## **Phase 4 — Pro**

16. Full end-to-end production pipeline
17. Use anomaly detection to detect:

    * unusually long time (idle tab open)
    * extremely low time (skimming)
18. Personalized recommendations:

    * If student spent high time → send revision
    * If less time → send micro-summaries
19. Run A/B test on redesigned chapters
20. Documentation + GitHub wiki

---

# ✅ **5. GitHub Folder Structure**

```
time-spent-analysis/
│
├── data/
│   ├── raw/
│   └── processed/
│
├── src/
│   ├── ingestion/
│   │   └── kafka_producer.py
│   ├── etl/
│   │   ├── compute_time_spent.py
│   │   ├── drop_off_detection.py
│   ├── analysis/
│   │   ├── chapter_time.ipynb
│   │   ├── difficulty_score.ipynb
│   └── ml/
│       └── difficult_chapter_predictor.py
│
├── airflow/
│   └── dags/
│       └── chapter_time_etl_dag.py
│
├── dbt/
│   └── models/
│       ├── chapter_summary.sql
│       └── chapter_analytics.sql
│
├── dashboard/
│   ├── powerbi/
│   └── streamlit_app.py
│
├── docs/
│   └── SRS.md
│
└── README.md
```

---

# ✅ **6. Beginner → Pro Curriculum (Optimized for Engagement Analytics)**

## **Module 1 — Beginner**

✔ Python Pandas
✔ SQL basics
✔ Data cleaning
✔ Time-based calculations
✔ Simple EDA

---

## **Module 2 — Intermediate**

✔ Data warehousing concepts
✔ Dimensional modeling for chapters/courses
✔ Airflow automation
✔ DBT transformations

---

## **Module 3 — Advanced**

✔ Spark for big chapter events
✔ Kafka streaming
✔ Advanced aggregations (window functions)
✔ Anomaly detection

---

## **Module 4 — Pro**

✔ ML for difficult chapter prediction
✔ Real-time insights
✔ API deployment
✔ Production pipeline architecture
✔ Monitoring & logging

---