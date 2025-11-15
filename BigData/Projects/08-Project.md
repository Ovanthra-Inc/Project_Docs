# 📘 Project: Student Mastery Score Engine (Adaptive Learning) — (Beginner → Pro)

A production-ready project to teach interns how to build a **Mastery Scoring Engine** that computes per-student, per-topic mastery levels and powers adaptive learning: personalized practice, quiz selection, study recommendations, and progress tracking.

Clear goals:

* Compute an interpretable **mastery score** (0–100% or 0–1) per student-topic (and rollups to subject/course).
* Support near-real-time updates from quiz/assignment/video interactions.
* Provide APIs and tables used by LMS for adaptive quizzes, recommendations, and parent/teacher insights.

---

# ✅ 1. SRS (Software Requirements Specification)

## 1.1 Project Overview

Create a pipeline + model that ingests student interactions (quiz attempts, practice problems, time spent, revisits, hints used) and outputs a **mastery score** per student-topic. Scores support:

* Personalized practice generation (focus on weak topics)
* Adaptive quiz difficulty selection
* Progress reports to students/parents/teachers
* Threshold-based alerts (e.g., mastery < 30% ⇒ intervention)

## 1.2 Functional Requirements

**FR-1 — Input Sources**

* Quiz attempts (is_correct, time_spent, attempts_count, timestamp, question_topic)
* Assignment submissions (score, timestamp)
* Video interactions (watch_pct, rewind_count)
* Practice sessions (accuracy, spaced repetitions)
* Tutoring/chatbot interactions (help_requests, resolved boolean)
* Course metadata (topic → chapter → subject hierarchy)

**FR-2 — Score Definition & Storage**

* Compute `mastery_score` ∈ [0,1] (or 0–100) for each `(student_id, topic_id, as_of_date)`
* Store in feature store and `student_mastery` gold table

**FR-3 — Update Cadence**

* Near-real-time updates (micro-batch, e.g., every 5–15 minutes) for active students
* Daily full recompute for historical backfills

**FR-4 — Explainability**

* Each score must be accompanied by a small explanation (top 3 contributing factors, e.g., low quiz accuracy, high time spent but low correctness, few practice attempts)

**FR-5 — API & Integration**

* Provide API endpoints:

  * `GET /mastery/{student_id}` (returns topics + scores + explanations)
  * `GET /mastery/{student_id}/{topic_id}/history`
* Provide event hooks for LMS to request adaptive content.

**FR-6 — Thresholds & Alerts**

* Configurable thresholds per school/grade to trigger notifications (teacher, parent, or student nudges)

**FR-7 — Monitoring & Data Quality**

* Validate required input signals; expose data quality metrics (completeness, freshness)
* Drift detection for feature distributions

**FR-8 — Security & Privacy**

* Pseudonymize student IDs in analytics, role-based access for dashboards, encrypt data at rest/in transit

## 1.3 Non-Functional Requirements

* **Scalability:** Support hundreds of thousands of students and tens of millions of events (batch & streaming).
* **Latency:** Near-real-time—most critical students update within 5–15 minutes.
* **Reliability:** 99.9% availability for scoring APIs.
* **Interpretability:** Scores auditable by teachers.
* **Cost-efficiency:** Tiered compute (real-time for active students, daily batch for archive).

## 1.4 Stakeholders

Students, Teachers, Parents, Product Managers, Data Engineers, ML Engineers, School Admins.

---

# ✅ 2. Conceptual Scoring Model & Algorithms

You can implement multiple candidate approaches; start simple and iterate.

## Option A — Rule-based Weighted Aggregation (fast to implement)

Mastery = weighted combination of normalized signals:

```
mastery = w1 * recent_quiz_accuracy
        + w2 * spaced_practice_success
        + w3 * recency_bonus
        + w4 * video_completion
        - w5 * hints_used_factor
```

Weights `w` tuned by domain experts or small grid search.

## Option B — Bayesian Knowledge Tracing (BKT)

Classic student modeling: tracks latent knowledge state per skill/topic using an HMM (learn, forget). Good for per-question sequential updates.

## Option C — Item Response Theory (IRT) + Elo-style updates

Model student ability and item difficulty; update ability score as student answers questions. Combine with decay and spaced repetition.

## Option D — Supervised ML Regression/Ranking

Use historical labeled mastery (teacher labels or proxy) to train regression (XGBoost or Spark GBT) with features aggregated over windows (7/30/90 days). Provide explainability via SHAP.

**Recommendation:** Start with Option A (rule-based) for bootstrapping, then implement BKT or Elo for more realistic sequence modeling. Add ML later for better calibration.

---

# ✅ 3. Dataset & Feature Design

## Raw events / signals (examples)

* `quiz_attempts`:

  * student_id, question_id, topic_id, is_correct, time_spent_sec, attempt_number, timestamp
* `assignments`:

  * student_id, assignment_id, topic_ids[], score_pct, timestamp
* `video_events`:

  * student_id, video_id, topic_id, watch_pct, watch_duration, timestamp
* `practice_sessions`:

  * student_id, topic_id, attempts, correct_count, timestamp
* `chatbot_interactions`:

  * student_id, topic_id, queries, resolved_count, timestamp
* `metadata`:

  * topic_id → difficulty_estimate, prerequisites, content_length

## Aggregated features (per student-topic, windowed)

Compute per window (e.g., last 7d, 30d, 90d):

* `q_attempts_7d`, `q_correct_7d`, `q_accuracy_7d`
* `avg_time_per_question_7d`
* `practice_attempts_30d`, `practice_success_rate_30d`
* `video_watch_pct_30d`
* `revisit_rate_30d` (fraction who revisit same topic)
* `hint_usage_rate`
* `last_activity_days_ago`
* `rolling_mastery_trend` (slope)
* `teacher_feedback_score` (if available)

## Output tables

### `student_mastery` (gold)

| Field         |      Type | Description                        |
| ------------- | --------: | ---------------------------------- |
| student_id    |    string | hashed                             |
| topic_id      |    string |                                    |
| mastery_score |     float | 0–1                                |
| score_source  |      enum | rule_based / bkt /ml               |
| as_of         | timestamp | scoring time                       |
| explanation   |      json | top contributing features & values |
| last_updated  | timestamp |                                    |

### `student_mastery_history`

Time series (for visualization & trend analysis).

---

# ✅ 4. Architecture Diagram (textual)

```
[Frontend / Quiz Engine / Video Player]  ---> Event Bus (Kafka)
                                              |
                                              v
                                  Raw Zone (S3 / MinIO)
                                              |
                                              v
                                     ETL / Feature Jobs
                         (Spark / PySpark / Airflow / DBT - compute aggregates)
                                              |
                                              v
                Feature Store / Aggregates (e.g., Delta Lake / BigQuery)
                                              |
            ┌─────────────────────────────────┴──────────────────────────┐
            v                                                            v
    Mastery Scoring Service (Streaming + Batch)                    Feature Store / ML
 (Rule-based microservice + BKT/Elo + ML scoring jobs)           (student_topic_features)
            |                                                            |
            v                                                            |
     student_mastery table (gold) <-------------------------------------┘
            |
            v
     API Layer (FastAPI) -> LMS / Dashboards / Notifications
            |
            v
      Teachers / Students / Parents UIs
```

---

# ✅ 5. Step-By-Step Tasks for the Intern

Divided into phases (beginner → pro) with concrete deliverables.

## Phase 0 — Setup (1–2 days)

* Clone repo template, install dependencies (Python, PySpark optional), set up local MinIO, DuckDB or small BigQuery sandbox.
* Review SRS and sample datasets.

## Phase 1 — Beginner (3–5 days) — Rule-based engine

1. **Data sampling:** Load sample `quiz_attempts` and `practice_sessions` CSV.
2. **Feature aggregation:** Write Python/Pandas script to compute per `(student, topic)` aggregated features for last 7 & 30 days.
3. **Rule-based scorer:** Implement `score = weighted_sum(features)` with configurable weights (YAML). Output `student_mastery` table and explanations (top contributing features).
4. **API:** Simple FastAPI endpoint to return mastery for a student.
5. **Dashboard:** Small Streamlit app showing mastery heatmap for one student.
   **Deliverable:** Working rule-based scorer, API, dashboard.

## Phase 2 — Intermediate (1–2 weeks) — Sequential models & robustness

6. **Implement BKT (or Elo variant):** Build per-topic BKT model to update mastery on each attempt.
7. **State store:** Store learner state (p_mastery) per student-topic (in Redis or state table) to support incremental updates.
8. **Micro-batch updates:** Hook Kafka consumer to update states/micro-batch recomputes.
9. **Full daily batch job:** Recompute all mastery scores daily (Spark/DBT job) and reconcile with incremental states.
10. **Explanations & thresholds:** Expand explanation module and mapping to strength/weakness tags.

## Phase 3 — Advanced (2–3 weeks) — ML and personalization

11. **Labeling:** Create labels (teacher-provided or proxy—e.g., future quiz accuracy) for supervised training.
12. **Train ML model:** Train regression model (XGBoost/Spark GBT) to predict future accuracy or a mastery ground-truth.
13. **Blend models:** Combine rule/BKT/ML outputs (ensembling or weighted average) and calibrate.
14. **A/B test:** Deploy to a subset of classes; measure improvement in learning outcomes (e.g., faster mastery gains).
15. **Monitoring:** Add drift detection on features & model performance; set retrain triggers.

## Phase 4 — Pro (production)

16. **CI/CD:** Automate training, validation, model deployment.
17. **Scaling:** Optimize Spark jobs, caching, and incremental feature computation.
18. **Governance:** Data lineage, access control, and PII handling.
19. **Teacher feedback loop:** UI for teachers to correct mastery; feed corrections to retraining pipeline.
20. **Documentation & Handoff:** Runbooks, API docs, and training materials.

---

# ✅ 6. GitHub Folder Structure

```
student-mastery-engine/
│
├── data/
│   ├── sample/
│   │   ├── quiz_attempts.csv
│   │   ├── practice_sessions.csv
│   │   └── video_events.csv
│
├── src/
│   ├── ingestion/
│   │   └── consumer.py        # kafka consumer to raw zone
│   ├── features/
│   │   └── build_student_topic_features.py
│   ├── scoring/
│   │   ├── rule_based.py
│   │   ├── bkt.py
│   │   ├── elo_updater.py
│   │   └── ml_scoring.py
│   ├── api/
│   │   └── fastapi_app.py
│   ├── serving/
│   │   └── scorer_service.py
│   ├── jobs/
│   │   ├── batch_recompute.py
│   │   └── microbatch_updater.py
│   ├── utils/
│   │   ├── hashing.py
│   │   └── explainers.py
│   └── tests/
│       └── test_scoring.py
│
├── infra/
│   ├── docker/
│   │   └── docker-compose.yml
│   └── k8s/
│
├── notebooks/
│   └── 01_mastery_rule_demo.ipynb
│
├── docs/
│   ├── SRS.md
│   └── api_docs.md
│
├── models/
│   └── ml_models/
│
├── requirements.txt
└── README.md
```

---

# ✅ 7. Example Scoring Logic (Rule-based) — Outline

1. Aggregate features for windows: 7d_acc, 30d_acc, practice_success_30d, last_activity_days, video_watch_pct, hints_rate.
2. Normalize each feature to [0,1] (min-max or logistic transform).
3. Apply weights: example

   ```
   mastery = 0.5*recent_accuracy
           + 0.2*practice_success
           + 0.15*revisit_rate
           + 0.1*video_completion
           - 0.05*hints_rate
   ```
4. Decay older evidence: apply exponential decay to older events so recent performance matters more.
5. Clamp to [0,1] and produce explanation: list top 3 features influencing score.

I'll generate working code if you want (Pandas or PySpark).

---

# ✅ 8. Monitoring, Evaluation & Metrics

* **Data-level:** completeness, freshness, null rates, schema drift.
* **Model-level:** correlation of predicted mastery vs next-week quiz accuracy (target), RMSE, calibration.
* **Business-level:** time-to-mastery reduction, reduction in teacher intervention, improved quiz pass rates.
* **Operational:** latency of updates, API error rate, job success/failure.

---

# ✅ 9. Beginner → Pro Curriculum (Specific to Mastery Engine)

## Week 1 — Foundations

* Domain: learning science basics (mastery, spaced repetition)
* Python, Pandas, SQL
* Event aggregation & time-windowing

## Week 2 — Rule-based & State

* Implement rule-based scoring
* Build simple API + dashboard
* Understand exponential decay and windowing

## Week 3 — Sequential Models

* Implement BKT/Elo per topic
* State management (Redis/state table)

## Week 4 — ML & Production

* Train ML model to predict next-quiz success
* Blend models, A/B tests, monitoring
* Build teacher feedback UI and integrate corrections

---

# ✅ 10. Deliverables for Intern

* Aggregation scripts that produce student-topic features (7/30/90d)
* Rule-based scorer + explanation output (CSV + API)
* A Streamlit demo dashboard (mastery heatmap, history chart)
* BKT/Elo incremental updater (simple implementation)
* Batch recompute job (Airflow DAG)
* Tests for scoring logic & data quality checks
* README + handoff docs

---
