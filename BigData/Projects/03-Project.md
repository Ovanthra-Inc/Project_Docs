
# 🎯 **Project: AI-Generated Quiz Performance Dashboard (Beginner → Pro)**

A real-world Big Data + Analytics project for your LMS where quizzes are AI-generated and student performance is analyzed deeply.

---

# ✅ **1. Detailed SRS (Software Requirement Specification)**

## **1.1 Project Overview**

Your LMS generates quizzes using AI (MCQ, subjective, difficulty-based).
This project analyzes:

* Student performance
* Difficulty distribution
* Time taken per question
* Per-topic mastery
* AI-generated question quality
* Predictive insights

Dashboard is for:
✔ Teachers
✔ Students
✔ Admin/Organization

---

# **1.2 Functional Requirements**

### **FR-1: Ingest AI Quiz Metadata**

Each AI-generated quiz must store:

* quiz_id
* question_id
* generated_by_model
* difficulty_level
* bloom_level
* question_type

### **FR-2: Track Student Attempts**

Collect attempt-level events:

* student_id
* quiz_id
* question_id
* selected_option
* correct_option
* is_correct
* time_spent_seconds
* attempt_timestamp

### **FR-3: Generate Metrics**

System must compute:

* Score
* Accuracy
* Topic-wise performance
* Difficulty-wise accuracy
* Average time per question
* Weak areas
* Strength areas
* Bloom-level mastery

### **FR-4: Dashboards**

Dashboards must display:

* Per quiz analytics
* Per student analytics
* Class-level analytics
* Topic-wise heatmap
* Difficulty pyramid
* Speed vs accuracy graph
* Top 5 difficult questions from AI

### **FR-5: Insights & Alerts**

Insights must include:

* “Student is weak in Algebra-Level-2”
* “AI-generated questions too difficult this week”
* “Quiz accuracy dropped by 25% since last version”
* “Students are taking too long on easy questions”

---

# **1.3 Non-Functional Requirements**

* **Scalability:** Handle 10M question-attempt records/day
* **Performance:** Dashboard load < 2 sec
* **Security:** Mask student identifiable info
* **Reliability:** No data loss in ingestion pipeline
* **Accuracy:** Quiz scoring must match model-generated answer keys

---

# **1.4 Stakeholders**

* Students
* Teachers
* Admins
* Data engineers
* AI team
* LMS product managers

---

# 🚀 **2. Dataset Design**

---

# **Table 1 — ai_quiz_metadata (raw)**

| Field         | Type                     | Description                  |
| ------------- | ------------------------ | ---------------------------- |
| quiz_id       | string                   | Unique quiz                  |
| question_id   | string                   | Unique Q                     |
| generated_by  | string                   | e.g., GPT-4, Llama 3, Claude |
| difficulty    | enum(easy, medium, hard) |                              |
| topic         | string                   | e.g., Algebra                |
| bloom_level   | enum(L1–L6)              |                              |
| question_type | mcq, true/false, fillups |                              |
| created_at    | datetime                 |                              |

---

# **Table 2 — student_attempts (raw)**

| Field              | Type           |
| ------------------ | -------------- |
| attempt_id         | string         |
| student_id         | string         |
| quiz_id            | string         |
| question_id        | string         |
| selected_option    | string         |
| correct_option     | string         |
| is_correct         | boolean        |
| time_spent_seconds | int            |
| attempt_timestamp  | datetime       |
| device_type        | mobile/desktop |

---

# **Table 3 — quiz_summary (processed)**

| Field                | Description       |
| -------------------- | ----------------- |
| quiz_id              | Quiz              |
| total_questions      | Count             |
| avg_difficulty       | Weighted score    |
| avg_time             | Mean time         |
| overall_accuracy     | (correct / total) |
| most_difficult_topic | Topic             |
| bloom_distribution   | JSON              |
| top_wrong_questions  | Array             |

---

# **Table 4 — student_performance_summary**

| Field                   | Description   |
| ----------------------- | ------------- |
| student_id              | User          |
| quiz_id                 | Quiz          |
| score_percent           | 0–100         |
| accuracy                | correct/total |
| avg_time                | seconds       |
| weak_topics             | JSON          |
| strengths               | JSON          |
| bloom_mastery           | JSON          |
| difficulty_accuracy_map | JSON          |

---

# **Table 5 — class_level_analytics**

| Field                  | Description |
| ---------------------- | ----------- |
| class_id               | Class       |
| quiz_id                | Quiz        |
| avg_score              | Aggregate   |
| avg_accuracy           | Avg correct |
| difficulty_trend       | JSON        |
| question_quality_score | ML-based    |

---

---

# 📊 **3. Architecture Diagram**

```
                 ┌────────────────────────┐
                 │     LMS Frontend       │
                 │  (Quiz Attempt Events) │
                 └──────────────┬─────────┘
                                │
                                ▼
                       ┌──────────────┐
                       │   Kafka Bus  │
                       └──────┬───────┘
                              │
                  ┌───────────▼────────────┐
                  │   ETL Pipeline (Airflow)│
                  │ PySpark / Pandas / DBT  │
                  └───────────┬────────────┘
                              │
                    ┌─────────▼───────────┐
                    │     Data Lake        │
                    │  S3 / MinIO / GCS    │
                    └─────────┬───────────┘
                              │
                    ┌─────────▼────────────┐
                    │   Data Warehouse      │
                    │ BigQuery / Redshift   │
                    └─────────┬────────────┘
                              │
       ┌──────────────────────▼────────────────────────┐
       │                BI Dashboard                   │
       │ PowerBI / Tableau / Looker / Metabase / Superset │
       └────────────────────────────────────────────────┘
```

---

# 📝 **4. Step-by-Step Tasks for the Intern**

---

## **Phase 1 — Beginner**

📌 Task 1: Explore dataset & create small sample CSV
📌 Task 2: Clean student_attempts dataset
📌 Task 3: Compute simple metrics

* Accuracy
* Time spent
* Score
  📌 Task 4: Plot
* Accuracy per quiz
* Time spent distribution
  📌 Task 5: SQL queries
* Top 10 difficult questions
* Average score per class
* Topic-wise accuracy

---

## **Phase 2 — Intermediate**

📌 Task 6: Build ETL script (Python Pandas)
📌 Task 7: Create Data Lake folder structure
📌 Task 8: DBT models

* quiz_summary
* student_performance_summary
  📌 Task 9: Build basic dashboards
* Quiz-level analytics
* Student-level insights

---

## **Phase 3 — Advanced**

📌 Task 10: Use PySpark for big datasets
📌 Task 11: Real-time quiz scoring ingestion
📌 Task 12: Build difficulty-trend graphs
📌 Task 13: Build ML model for question difficulty prediction
📌 Task 14: Personalized recommendation logic

---

## **Phase 4 — Pro**

📌 Task 15: Design complete real-time analytics system
📌 Task 16: Build complete dashboard using Metabase/PowerBI
📌 Task 17: Integrate dashboard into LMS (iframe or API)
📌 Task 18: Add anomaly alerts

* Sudden drop in accuracy
* Questions with high wrong-rate
  📌 Task 19: Question Quality Scoring ML model
  📌 Task 20: Full documentation + GitHub wiki

---

# 📁 **5. GitHub Folder Structure**

```
ai-quiz-performance-dashboard/
│
├── data/
│   ├── raw/
│   └── processed/
│
├── src/
│   ├── ingestion/
│   │   └── kafka_consumer.py
│   ├── etl/
│   │   ├── quiz_summary.py
│   │   ├── student_performance.py
│   ├── analysis/
│   │   ├── accuracy_analysis.ipynb
│   │   ├── topic_trends.ipynb
│   └── ml/
│       └── question_difficulty_model.py
│
├── airflow/
│   └── dags/
│       └── quiz_etl_dag.py
│
├── dbt/
│   └── models/
│       ├── quiz_summary.sql
│       ├── student_performance.sql
│       └── class_analytics.sql
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

# 📘 **6. Beginner → Pro Curriculum (for your Intern)**

---

## 🔰 **Beginner (Week 1–2)**

* Python basic
* Pandas
* SQL basics
* EDA
* Data cleaning
* Accuracy/time calculations

---

## ⚙️ **Intermediate (Week 3–5)**

* ETL pipelines
* Airflow
* DBT
* Data modeling (star schema)
* BI dashboard basics

---

## 🚀 **Advanced (Week 6–8)**

* Kafka ingestion
* PySpark
* Real-time analytics
* Complex SQL (window functions)
* ML basics
* Feature engineering

---

## 🧠 **Pro (Week 9–12)**

* ML difficulty scoring
* LLM scoring of question quality
* Recommendation engine
* Production-grade monitoring
* API development
* Scaling data pipeline
