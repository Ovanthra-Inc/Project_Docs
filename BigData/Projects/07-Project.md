# 📘 Project: Chapter Difficulty Prediction (Spark MLlib) — (Beginner → Pro)

A production-ready project to teach interns how to build a **chapter-level difficulty prediction** model using **Apache Spark (PySpark) + Spark MLlib**.
Goal: predict whether a chapter is *easy / medium / hard* (or a continuous difficulty score) using engagement, performance, and content features — to help the LMS adapt content, flag chapters for revision, and surface teacher insights.

---

# ✅ 1. Detailed SRS (Software Requirement Specification)

## 1.1 Project Overview

Predict chapter difficulty using historical student interaction data (time spent, quiz accuracy, attempts, drop-off, etc.). Output: difficulty label (easy/medium/hard) or numeric difficulty score (0–1). Serve predictions to dashboards, recommendation engine, and content team.

## 1.2 Functional Requirements

* **FR-1: Input data** — Accept aggregated per-chapter signals (chapter_id, course_id, timestamp window, aggregated features).
* **FR-2: Labeling** — Provide labeled training data from historical metrics (rule-based or teacher-labeled).
* **FR-3: Model training** — Use Spark MLlib pipelines for featurization, training, tuning, and cross-validation.
* **FR-4: Model evaluation** — Produce classification metrics (precision, recall, F1), confusion matrix; or regression metrics (RMSE, MAE, R²) for continuous score.
* **FR-5: Model explainability** — Provide feature importance and per-chapter explanations (global and local).
* **FR-6: Batch prediction** — Score chapters nightly and store results in `chapter_difficulty` table.
* **FR-7: API access** — Predictions accessible via REST API for dashboard display.
* **FR-8: Monitoring** — Model drift detection, data-quality checks, and retraining triggers.
* **FR-9: Reproducibility** — Pipelines versioned (code + model), reproducible training runs, and model artifacts stored.

## 1.3 Non-Functional Requirements

* **Scalability:** Handle thousands of chapters and millions of aggregated rows using Spark.
* **Latency:** Training can be batch (daily/weekly). Scoring latency per batch < 10 minutes.
* **Reliability:** Retrain if data drift detected; logs for training and scoring.
* **Security:** No PII in features; only chapter-aggregates and hashed IDs.
* **Interpretability:** Provide human-understandable reasons for predicted difficulty.

## 1.4 Stakeholders

Product, Content Team, Teachers, Data Engineers, ML Engineers, Analysts.

---

# ✅ 2. Dataset & Labeling Design

## 2.1 Candidate Feature Set (per chapter, per time window — e.g., last 30 days)

|               Feature | Description                                      |
| --------------------: | ------------------------------------------------ |
|            chapter_id | chapter identifier                               |
|             course_id | course identifier                                |
|        avg_time_spent | average seconds spent per visit                  |
|     median_time_spent | robust central tendency                          |
|      total_time_spent | total seconds across users                       |
|          visits_count | total visits                                     |
|       unique_students | number of distinct students                      |
|     avg_quiz_accuracy | average correctness for chapter questions        |
|      median_quiz_time | median time per question                         |
| attempts_per_question | avg attempts                                     |
|          dropoff_rate | fraction leaving before 50% content              |
|      scroll_depth_avg | average scroll/completion %                      |
|          revisit_rate | fraction who revisit chapter                     |
|         help_requests | number of AI-doubt queries for chapter           |
|       video_watch_pct | if video: avg % watched                          |
|       forum_questions | number of forum/doubt posts                      |
|   difficulty_feedback | teacher or student feedback score (if available) |
|        content_length | text length / number of slides / videos          |
|  embedding_complexity | optional — content embedding norm or length      |
|          week_of_term | temporal covariate                               |

## 2.2 Labeling strategies

Choose one or combine:

1. **Rule-based labeling (good for bootstrapping)**

   * `hard` if avg_quiz_accuracy < 50% **AND** avg_time_spent > threshold AND dropoff_rate > 0.3
   * `easy` if avg_quiz_accuracy > 85% AND avg_time_spent < threshold
   * `medium` otherwise

2. **Teacher-labeled data**

   * Periodic manual labels from content experts (best quality).

3. **Continuous score (preferred long-term)**

   * Define difficulty score in [0,1] as weighted combination of inverted accuracy, normalized time_spent, and dropoff_rate.

Note: start with rule-based labels to create the initial training set, then refine with teacher labels.

## 2.3 Data partitions

* Training/validation/test split by time or by chapters (time-based split recommended to avoid leakage).
* Maintain a holdout set of recent weeks for final evaluation.

---

# ✅ 3. Architecture Diagram (textual)

```
[Raw Events] -> [ETL: Bronze→Silver→Gold] -> [Chapter Aggregates Table]
                                           |
                                           v
                                  [Feature Store / Aggregates]
                                           |
                                  [Spark MLlib Training Job]
                                           |
                       ┌───────────────────┴───────────────────┐
                       v                                       v
               [Model Artifacts (MLflow/MinIO)]        [Batch Scoring Job]
                       |                                       |
                       v                                       v
      [Model Registry / Versioning & Monitoring]        [chapter_difficulty table]
                       |                                       |
                       v                                       v
                [Dashboard / API / Teacher UI]         [Retrain Triggers (drift)]
```

---

# ✅ 4. Step-by-Step Tasks for the Intern (Phased)

## Phase 0 — Prep (1–2 days)

* Understand the SRS, dataset examples, and Spark basics.
* Setup environment: PySpark, MLflow (optional), Docker, MinIO or S3, Jupyter.

## Phase 1 — Beginner (3–5 days)

1. **Data exploration** — load chapter aggregates into Spark DataFrame, compute distributions.
2. **Label creation** — implement rule-based labeling script and create labeled dataset.
3. **Baseline model** — train a simple logistic regression (classification) or linear regression (score).
4. **Evaluation** — compute accuracy, precision, recall, F1 (classification) or RMSE, MAE (regression).
5. **Feature importance** — use model coefficients or tree feature importances.

**Deliverable:** notebook + baseline model saved.

## Phase 2 — Intermediate (1–2 weeks)

6. **Feature engineering pipeline** — Spark ML Pipeline: Imputer -> StringIndexer -> OneHotEncoder -> VectorAssembler -> Scaler.
7. **Try tree-based models** — RandomForestClassifier/Gradient-Boosted Trees (GBT) in MLlib.
8. **Hyperparameter tuning** — use CrossValidator/TrainValidationSplit with ParamGrid.
9. **Model explainability** — extract feature importances; for local explanations export to pandas sample and use SHAP (optional: convert Spark sample to pandas).
10. **Model evaluation with time-based holdout** — ensure model generalizes to future data.

**Deliverable:** tuned model, evaluation report, feature importance plot.

## Phase 3 — Advanced (2–3 weeks)

11. **Pipelineize training** — wrap training in a reproducible Spark job (submitable via spark-submit), track with MLflow.
12. **Batch scoring job** — nightly Spark job that scores all chapters, writes `chapter_difficulty` to Data Warehouse.
13. **Drift detection** — compare current prediction distribution & feature distributions to training baseline; raise retrain flag.
14. **Calibration & thresholds** — map regression score to easy/medium/hard thresholds using validation set.
15. **A/B testing** — serve difficulty predictions to a subset of schools and measure teacher feedback / downstream impact.

**Deliverable:** deployed batch scoring + retrain pipeline.

## Phase 4 — Pro (production)

16. **CI/CD for models** — automated retraining pipelines, tests, and deployment.
17. **Monitoring & alerts** — model performance pages, data quality, and anomaly alerts.
18. **Human-in-the-loop** — UI for teachers to correct difficulty labels; integrate feedback into next training round.
19. **Online inference (optional)** — low-latency microservice for on-demand scoring.

---

# ✅ 5. Evaluation Metrics & Validation

### For classification (easy/medium/hard)

* Accuracy, Precision, Recall, F1 per class
* Confusion matrix (inspect common confusions)
* Macro-F1 (if classes imbalanced)
* AUC (if binary or one-vs-rest)

### For regression (continuous score)

* RMSE, MAE, R²
* Calibration plot (predicted vs observed difficulty metric)
* Spearman correlation vs teacher labels

### Robust validation

* Time-based holdout (train on older weeks, test on recent weeks)
* Per-course cross-validation to avoid course-specific leakage

---

# ✅ 6. Feature Engineering & Modeling Tips

* Normalize time features (log-transform for heavy tails).
* Use robust statistics (median) to avoid outliers.
* Create interaction features: `avg_time_spent / avg_quiz_time`.
* Use categorical embedding for course/subject (or one-hot if few categories).
* Handle missing values with sensible defaults and imputation.
* Balance classes (SMOTE not native in Spark — sample weights or undersampling).
* Start with interpretable models (LR, RF) before complex ensembles.

---

# ✅ 7. GitHub Folder Structure

```
chapter-difficulty-prediction/
│
├── data/
│   ├── raw/                    # sample aggregates and raw features
│   └── labeled/
│
├── notebooks/
│   ├── 01_explore_aggregates.ipynb
│   ├── 02_labeling_and_baseline.ipynb
│   └── 03_model_tuning.ipynb
│
├── src/
│   ├── features/
│   │   └── build_features.py
│   ├── labeling/
│   │   └── labeler.py
│   ├── training/
│   │   └── train_spark.py
│   ├── scoring/
│   │   └── batch_score.py
│   ├── serving/
│   │   └── api_server.py   # optional; for on-demand scoring
│   └── utils/
│       └── io.py
│
├── pipelines/
│   └── train_pipeline.yaml   # or airflow/dag for orchestration
│
├── models/
│   └── archived/             # saved model artifacts (MLflow/MinIO)
│
├── tests/
│   └── test_feature_build.py
│
├── docs/
│   └── SRS.md
│
├── Dockerfile
├── requirements.txt
└── README.md
```

---

# ✅ 8. Deliverables (what the intern should produce)

* EDA notebook showing feature distributions and target distribution.
* Labeling script with rationale and examples.
* Baseline model (logistic regression or linear regression).
* Tuned tree-based model with evaluation metrics.
* Spark training job (`train_spark.py`) and batch scoring job.
* Model artifact logged (MLflow) with version.
* README with run instructions + sample data.
* Short demo dashboard (or CSV) of `chapter_difficulty` table.

---

# ✅ 9. Example Spark MLlib Pipeline (high-level steps)

1. Read chapter aggregates (Spark DataFrame).
2. Apply labeler (UDF or join with labeled table).
3. Split train/validation/test (time-based).
4. Build `Pipeline(stages=[imputer, indexers, encoders, assembler, scaler, estimator])`.
5. Use `CrossValidator` or `TrainValidationSplit` with `ParamGridBuilder`.
6. Fit model, log metrics, save model.
7. Batch score by loading model and writing predictions.

(If you want, I can generate `train_spark.py` starter code next.)

---

# ✅ 10. Beginner → Pro Curriculum (ML on Spark)

## Module 1 — Spark Basics (1 week)

* PySpark DataFrame API
* Spark SQL
* Reading/writing Parquet

## Module 2 — MLlib Fundamentals (1 week)

* MLlib Pipelines, Transformers, Estimators
* Feature engineering in Spark
* Basic models: LogisticRegression, RandomForest, GBT

## Module 3 — Model Tuning & Validation (1 week)

* ParamGrid, CrossValidator
* Time-based validation strategies
* Model explainability (feature importances)

## Module 4 — Productionization (1–2 weeks)

* MLflow integration
* Batch scoring jobs & orchestration (Airflow)
* Monitoring, drift detection, and retraining triggers

## Module 5 — Advanced (optional)

* Convert Spark model to ONNX (if needed)
* Local explanations (SHAP) on sampled data
* Online inference service

---