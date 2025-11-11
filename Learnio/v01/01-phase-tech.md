# Full Updated Tech Stack (MVP-ready)

## 1 — Architecture & High-level decisions

* **Architecture:** Multi-tenant SaaS (row-level isolation)
* **Design approach:** Modular monolith for MVP
* **API paradigm:** REST (simple, fast, predictable)
* **Storage approach:** Relational primary store + object storage

  * Primary relational DB: Google Cloud SQL (Postgres)
  * Object storage: Google Cloud Storage (GCS)
* **Automation layer:** Background jobs + event-based triggers (BullMQ + Redis Pub/Sub)
* **Infrastructure posture:** Cloud-native (GCP), CDN, monitoring, auto-scaling

---

## 2 — Frontend (Creator, Student, Admin UI)

* **Framework:** Next.js (TypeScript)
* **Routing:** Next.js App Router (dynamic routes for channel pages)
* **UI styling:** Tailwind CSS + shadcn/ui (component primitives)
* **State management:** Redux Toolkit
* **Forms & validation:** Zod (schema validation)
* **Video playback:** ReactPlayer (recorded content)
* **Community (client):** Socket.io client (real-time events & presence)
* **Certificate viewer / export:** HTML template → headless renderer (HTML → PDF) for preview & download
* **Theming:** Channel styling tokens (theme metadata per channel used by CSS variables / Tailwind config)

**Notes:** Next.js gives SSR/SSG for public channel landing pages (important for SEO and sharing).

---

## 3 — Backend & API

* **Primary backend framework:** NestJS (Node + TypeScript) — single monolith handling all API routes for MVP
* **API layer:** REST endpoints (JSON API)
* **API Gateway / reverse proxy:** **NGINX** (reverse proxy, TLS termination, rate limiting, static asset handling)

  * NGINX sits in front of Cloud Run services (or load balanced endpoints) to offload TLS and static caching policies.
* **Authentication:** JWT + Refresh Tokens (custom email/password flows)
* **RBAC:** Custom policy-based access in application layer (channel scoped permissions)
* **Session management:** Redis (session store + Socket.io adapter)
* **Server-side real-time:** Socket.io server + Redis adapter (for scaling across instances)
* **File upload handling (server):** Signed, direct uploads to GCS (server issues signed upload URLs or presigned policy)
* **Audit & activity logs:** Append-only audit log table (actor, action, target, metadata, timestamp)

**Notes:** Keep tenant scoping (channel_id) checked on each request and DB query.

---

## 4 — Persistence & Storage

* **Relational DB:** Google Cloud SQL (Postgres) — all structured data: users, channels, courses, batches, enrollments, messages metadata, events table, audit logs.
* **Object storage:** Google Cloud Storage (GCS) — videos (recordings/MP4), thumbnails, certificates (PDF), large resource files.
* **Cache & Pub/Sub:** Redis (managed Redis) — session store, Socket.io adapter, Redis Pub/Sub for internal events.
* **Search:** Elasticsearch — full-text search for channels, courses, community posts.
* **Analytics event store (MVP):** Lightweight event table in Postgres (event_type, user_id, channel_id, metadata, timestamp). Can export to BigQuery for advanced analytics later.

---

## 5 — Video / Live Class Stack

* **Live provider:** (Your chosen provider — use provider that supports recording + webhooks; ensure it can produce HLS or MP4 recordings.)
* **Recording pipeline:** Live provider records → provider webhook → upload recording (MP4 or HLS bundle) to GCS → create lesson entry with metadata (duration, thumbnail).
* **Playback for MVP:** Serve MP4s from GCS via signed URLs + Cloud CDN (Cloud CDN in front of GCS). Defer full HLS transcoding (Mux) until needed.
* **In-session features (MVP):** Rely on live provider for screenshare and in-session chat; optionally mirror session events into platform messages for persistence.
* **Attendance tracking:** Use live provider webhooks (join/leave) + client heartbeat for recorded watch progress (onPlay/onPause/heartbeat).

**Notes:** Ensure recordings are stored with proper access controls and lifecycle rules (e.g., retention for free tiers).

---

## 6 — Real-time & Messaging

* **Socket layer:** Socket.io (server + client) for real-time community & in-app notifications.
* **Adapter & pub/sub:** Redis adapter + Redis Pub/Sub for multi-instance scaling.
* **Message persistence:** Store messages/threads in Postgres (for moderation, export, search). Use Redis for presence and ephemeral state.
* **Moderation tools:** Flagging, soft-deletes, admin moderation queue.

**Notes:** This implementation balances control (custom RBAC) and cost (no vendor lock-in).

---

## 7 — Background Jobs, Scheduling & Automation

* **Job queue:** BullMQ (Redis-backed) — processing heavy tasks, background jobs, and repeatable jobs.
* **Task scheduler:** BullMQ repeatable jobs + Cloud Scheduler fallback for critical tasks (health checkpoint or webhook-triggered jobs).
* **Common jobs:** certificate generation, email reminders, nightly analytics aggregation, resource archiving, recording post-processing.
* **Event bus:** Redis Pub/Sub for internal event routing and lightweight event-driven behavior.

---

## 8 — Payments & Billing

* **Payment processor:** Stripe (Connect) — subscriptions for channels, checkout for student enrollments, payout flows for creators.
* **Billing model support:** Monthly subscription + transaction fees + optional plugin/marketplace billing.
* **Dispute handling:** Integrate Stripe webhooks to capture refunds/chargebacks into admin workflows.

---

## 9 — Certificates & Share/Tracking

* **Certificate generation:** HTML templates → headless renderer (server side) → PDF output. Store PDFs in GCS.
* **Verification & tracking:** Generate unique short verification hash/URL per certificate. Track clicks/visits and shares as events in event table.
* **Share options:** Provide share actions (LinkedIn, X, download) that log share events.

---

## 10 — CDN, Delivery & Security

* **CDN:** Cloud CDN (GCP) in front of GCS for static/video asset distribution. Optional Cloudflare for WAF, DNS, and additional caching.
* **Signed URLs:** Use signed GCS URLs for secure, time-limited access to protected videos/resources.
* **TLS / Security headers:** Terminate TLS at NGINX/CDN; add HSTS, CSP, and rate limiting.
* **API security:** Rate limiting (NGINX / application), CSRF protections where needed, input validation (Zod), RBAC checks on every write.
* **Audit logging:** Append-only logs in Postgres with export capability.
* **Backups:** Automated Cloud SQL backups + GCS lifecycle & backup policies.

---

## 11 — Observability & Monitoring

* **Metrics & monitoring:** Prometheus (metrics) + Grafana (dashboards) for infrastructure & application metrics.
* **Logging & error tracking:** Centralized logs (Cloud Logging / ELK) + Sentry for error tracking.
* **Uptime & alerts:** Alerting rules in Grafana/Prometheus for critical incidents (queue backlog, failed jobs, high error rate).

---

## 12 — CI/CD, IaC & Deployment

* **Frontend hosting:** Vercel (Next.js) — automatic builds & preview deployments.
* **Backend hosting:** Google Cloud Run (containerized NestJS services) — autoscaling managed. NGINX as reverse proxy in front if desired (containerized or managed).
* **Database hosting:** Google Cloud SQL (Postgres).
* **CI/CD:** GitHub Actions (build/test/deploy pipelines).
* **Infrastructure as code:** Terraform (manage Cloud Run, IAM, GCS buckets, Cloud SQL, Redis, Cloud CDN).
* **Domain & DNS:** Cloudflare recommended for DNS, WAF, and global CDN features (GoDaddy ok for domain registration; manage DNS through Cloudflare).

**Notes:** Cloud Run + Cloud SQL + GCS aligns with chosen cloud provider and minimizes operational overhead.

---

## 13 — Analytics & Dashboards (Creator-facing)

* **MVP analytics source:** Aggregate events from event table in Postgres → generate lightweight in-app charts + CSV export.
* **Creator KPI charts:** watch time, lesson completion, attendance per batch, resource downloads, certificate counts, revenue.
* **Future export / warehouse:** Export to BigQuery for heavier analysis / BI (Metabase / Looker Studio) when needed.

---

## 14 — Additional components & responsibilities

* **Search indexing:** Elasticsearch indexing for course/channel/community search. Keep asynchronous indexing pipeline to avoid blocking writes.
* **Spam & abuse protection:** Rate limits, moderation queue, content filters (future AI-based moderation).
* **GDPR & privacy:** Data export & deletion APIs, privacy policy, consent for tracking/sharing.
* **Testing & QA:** Unit tests, integration tests, staging environment on Cloud Run.

---

## 15 — Summary (single-page snapshot)

**Frontend:** Next.js (TS), Tailwind, shadcn/ui, Redux Toolkit, Zod, ReactPlayer, Socket.io client
**Backend:** NestJS (Node+TS) monolith, REST API, NGINX reverse proxy
**DB & Storage:** Google Cloud SQL (Postgres), Google Cloud Storage (GCS)
**Real-time & Cache:** Socket.io, Redis (session store, pub/sub, BullMQ adapter)
**Jobs & Scheduling:** BullMQ (repeatable jobs) + Cloud Scheduler fallback
**Live / Recording:** Live provider (webhooks → recordings → GCS), MP4 playback via signed URLs + Cloud CDN
**Search & Analytics:** Elasticsearch (search), Postgres events (analytics), BigQuery for future warehousing
**Payments:** Stripe Connect
**Observability:** Prometheus + Grafana, Sentry for errors
**CI/CD & IaC:** GitHub Actions, Terraform
**DNS/WAF:** Cloudflare (recommended)
**Certificates:** HTML → headless PDF renderer → stored on GCS, verification hash/table recorded

---

## Quick operational recommendations (for your team)

1. **Enforce tenant scoping** as a pattern — every DB write/query must include `channel_id`.
2. **Instrument the player early** (onPlay/onPause/heartbeat) so analytics are meaningful from day one.
3. **Use provider webhooks** for live session events to ensure accurate attendance logs.
4. **Keep message persistence** (Postgres) even if using Socket.io — needed for moderation and search.
5. **Build a small admin moderation UI** from the start (flagged messages, user bans, content takedown).
6. **Centralize configuration** for channel branding (theme tokens) so landing pages are auto-generated.
7. **Protect certificate endpoints** and record verification access for anti-fraud.
