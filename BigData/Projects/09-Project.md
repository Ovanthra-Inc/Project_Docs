# 🎯 Project: **Real-Time Student Engagement Dashboard**

(Streaming + Analytics + Dashboard — Beginner → Pro)

This project teaches interns **real-time big data analytics**, using streaming events (clicks, video interactions, quiz events) to build a **live engagement dashboard** for your AI-first LMS platform.

This dashboard is used by:

* Teachers → real-time class monitoring
* Admins → engagement trends across school
* LMS → adaptive nudges & notifications

---

# ✅ **1. SRS — Software Requirements Specification**

## **1.1 Project Summary**

Build a real-time dashboard that shows how active students are on the platform *right now* by ingesting live events (clickstream, video interactions, quiz progress) using a streaming engine (Kafka → Spark Streaming → Redis/Postgres) and displaying in a UI (Next.js or Streamlit).

## **1.2 Functional Requirements**

### **FR-1: Real-time event ingestion**

Events include:

* Page views
* Button clicks
* Video start/pause/stop
* Quiz start/question navigation
* Assignment submission
* Idle → Active transitions

Events pushed from frontend → Kafka.

### **FR-2: 3 Core Engagement KPIs**

Dashboard must show updated values every **3–5 seconds**:

1. **Active Students Right Now**
2. **Top Active Subjects/Chapters**
3. **Live Engagement Score (0–100)**

### **FR-3: Dashboard Widgets**

* Live Active Users Counter
* Engagement Trend (last 10 minutes)
* Per-Class Activity Heatmap
* Session Start/End Logs
* Realtime Events Feed (latest 20 events)
* Time Spent per Module (live)
* Device/Browser Summary

### **FR-4: API Endpoints**

* `GET /engagement/active_students`
* `GET /engagement/trend?window=10min`
* `GET /engagement/events/latest`
* `GET /engagement/class/{class_id}/heatmap`

### **FR-5: Data Freshness Requirements**

* Max dashboard latency: **< 5 seconds**
* Max Kafka → Stream processing delay: **< 2 seconds**

### **FR-6: Alert System**

Generate alerts if:

* Active students drop suddenly
* Video engagement < threshold
* Mulitple idle users > 30%

### **FR-7: Scalability**

Must scale to:

* 20,000+ students
* 200–500 events/second

---

# ✅ **2. Dataset Design**

You need two datasets:

---

## **A. Raw Clickstream Events (Kafka Topic: `clickstream_events`)**

| Field      | Type   | Description                                |
| ---------- | ------ | ------------------------------------------ |
| event_id   | string | uuid                                       |
| student_id | string | hashed                                     |
| class_id   | string | 1–12                                       |
| subject_id | string | e.g., MATH, SCI                            |
| chapter_id | string | optional                                   |
| event_type | enum   | click, page_view, quiz_action, video_event |
| event_name | string | “video_play”, “quiz_question_next”, etc.   |
| timestamp  | bigint | epoch ms                                   |
| metadata   | json   | device info, location, etc.                |

---

## **B. Processed Engagement Metrics (Redis/DB Table: `student_live_metrics`)**

| Field            | Type   | Description                  |
| ---------------- | ------ | ---------------------------- |
| student_id       | string | hashed                       |
| last_active_ts   | bigint | epoch                        |
| current_activity | string | reading video/quiz/dashboard |
| engagement_score | float  | 0–1                          |
| device           | string | mobile/desktop               |

---

---

# ✅ **3. Real-Time Architecture (Text Diagram)**

```
                [Student Browser / App]
                     | (events)
                     v
                 Kafka Producer JS
                     |
                     v
                Kafka Cluster
        (clickstream_events topic)
                     |
                     v
             Streaming Processor
          (Spark Streaming / Flink)
       - Update engagement metrics
       - Compute active student count
       - Produce dashboard KPIs
                     |
                     v
              Redis (real-time)
          - active_users_count
          - 10-min engagement trend
          - latest events list
                     |
                     v
         FastAPI / Express.js API Layer
                     |
                     v
       Real-Time Dashboard (Next.js)
         - WebSockets or SSE updates
```

---

# ✅ **4. Engagement Score Formula (Simple Version)**

```
engagement = 
    w1 * recency_factor           (last activity)
  + w2 * event_intensity_factor  (video > quiz > click)
  + w3 * session_duration_factor
  + w4 * streak_factor
```

0–1 normalized → multiplied by 100 in UI.

---

# ✅ **5. Step-by-Step Tasks for Intern**

## **Phase 1 — Beginner (3–4 Days)**

### 🎯 Output: Local Mock Kafka + simple processor

1. Build frontend event tracker (JS function).
2. Send mock events to Kafka topic (`clickstream_events`).
3. Write Python script to read Kafka events (console consumer).
4. Create simple Redis keys:

   * `active_students_count`
   * `latest_events`
5. Build a basic dashboard skeleton (Next.js or Streamlit).

---

## **Phase 2 — Intermediate (1 Week)**

### 🎯 Output: Working real-time dashboard

6. Implement Spark Streaming job to aggregate events every 5 seconds.
7. Compute engagement metrics and store in Redis hash:

   * `engagement:<student_id>`
8. Build Dashboard widgets:

   * Active now
   * Trendline (10-min)
   * Class heatmap
9. Add WebSocket updates in UI.
10. Create FastAPI endpoints for:

    * `/engagement/active`
    * `/engagement/trend`

---

## **Phase 3 — Advanced (2 Weeks)**

### 🎯 Output: Production-like system with analytics

11. Add device-level analytics.
12. Add ML model to predict who is about to turn idle.
13. Add alerting system (Slack/Email).
14. Multi-class dashboard (class-wise, grade-wise).
15. Data quality checks (missing fields, delayed events).

---

## **Phase 4 — Pro Level (Optional)**

### 🎯 Output: Scalable, fully-fledged system

16. Horizontal scaling for Kafka & Spark.
17. Event time vs Processing time windowing (watermarks).
18. Implement Flink or Kafka Streams version.
19. Add API rate limits & caching.
20. Deploy to Kubernetes with autoscaling.

---

# ✅ **6. GitHub Folder Structure**

```
real-time-student-engagement/
│
├── data/
│   └── sample_events.json
│
├── kafka/
│   └── docker-compose.yml
│
├── producer/
│   └── js_frontend_producer.js
│
├── consumer/
│   └── spark_streaming_processor.py
│
├── api/
│   └── fastapi_server.py
│
├── redis/
│   └── redis_init.sh
│
├── dashboard/
│   └── nextjs-app/
│
├── utils/
│   └── engagement_score.py
│
├── docs/
│   ├── SRS.md
│   ├── architecture.md
│   ├── api_docs.md
│
└── README.md
```

---

# ✅ **7. Dashboard Widgets (Intern Build List)**

### Must-have:

✔ Active students counter (real-time)
✔ Recent events (scrolling list)
✔ 10-minute engagement trend
✔ Per-class engagement table
✔ Real-time heatmap (class × chapter)

### Good-to-have:

• Device usage pie chart
• Browser usage chart
• Lag detection

---

# ✅ **8. Beginner → Pro Learning Curriculum (Specific to Real-Time Dashboard)**

## Week 1 — Basics

* Kafka fundamentals
* JSON event design
* Python Kafka Consumer
* Intro to Redis (publish/subscribe, sorted sets)

## Week 2 — Streaming Analytics

* Spark Streaming or Kafka Streams
* Windowing & aggregation
* Engagement scoring model
* Trendline computations

## Week 3 — Real-Time Dashboard

* Next.js basics
* WebSockets vs SSE
* Live charts (Recharts)
* Heatmap & counters
* FastAPI backend

## Week 4 — Production Concepts

* Metrics & monitoring (Prometheus)
* Retry & replay
* Offset handling
* Kubernetes deployment
* Data quality & observability

---