# Architecture Decision Records (ADRs)

## ADR-001: Task Orchestration with Temporal/Airflow

**Problem/Context**  
[SRS §4.1 FR-003, §5 NFR-R-01, Stories: RUN_ID tracking, idempotent retries]  
Pipeline must track multi-stage runs (preproc → segment → metrics), retry transient failures, and guarantee idempotence. Manual orchestration risks duplicates and lost provenance.

**Options**  
1. **Custom scheduler** (Python + Celery + Redis): full control, low overhead.  
2. **Temporal** (durable execution): native retry, state persistence, versioning.  
3. **Airflow** (DAG-based): mature, UI, integrations.

**Trade-offs**  
- Custom: Fast iteration but no battle-tested idempotency; higher bug surface (NFR-R-01 at risk).  
- Temporal: Strong guarantees, deterministic replays; steeper learning curve; Go runtime overhead.  
- Airflow: Rich UI, broad community; DAG re-parsing can be slow; less granular state control.

**Decision**  
Adopt **Temporal** for critical pipelines; **Airflow** for scheduled exports.  
Temporal's deterministic replay and built-in retries directly satisfy NFR-R-01 (MTTR <30 min) and FR-003 (immutable manifest). Airflow handles time-triggered exports (Story: cohort exports).

**Revisit Criteria**  
If run complexity stays <5 stages and team lacks Go expertise, migrate to Airflow-only with custom idempotency keys.

**Links**  
C4: Orchestrator container | Stories: RUN_ID tracking, idempotent retries, cohort exports | NFR-R-01, NFR-R-02

---

## ADR-002: Model Registry with MLflow + Promotion Gates

**Problem/Context**  
[SRS §4.7 FR-060..FR-062, Stories: register model card, canary deploys, drift alerts]  
ML Engineer must version models, attach cards (data hash, metrics, risks), deploy canaries, and detect drift. Ad-hoc versioning breaks reproducibility.

**Options**  
1. **MLflow**: open-source, model cards, REST API, no native A/B.  
2. **Seldon Core**: K8s-native canary, but limited card support.  
3. **Custom Git-LFS + DB**: minimal tooling, full control, high maintenance.

**Trade-offs**  
- MLflow: Proven for research; weak canary/shadow traffic; ECE drift needs custom job.  
- Seldon: Strong deployment patterns; overkill for non-k8s; card format immature.  
- Custom: No lock-in; reimplements 80% of MLflow; team bandwidth.

**Decision**  
**MLflow** for registry + cards; **shadow traffic in API Gateway** (feature flags route 1% traffic to candidate). Weekly cron computes population stats and ECE; alerts if NFR-A-02 violated (calibration >0.05).

**Revisit Criteria**  
If canary deploys exceed 10/month or drift logic becomes complex, evaluate Seldon or KServe.

**Links**  
C4: Model Registry, Segmenter | Stories: register model, canary, drift | FR-060, FR-061, FR-062, NFR-A-02

---

## ADR-003: S3-Compatible Object Store for Artifacts

**Problem/Context**  
[SRS §2.3, §4.1 FR-004, §6 DR-03, Stories: provenance manifest, PHI encryption]  
Volumes (GB-scale), masks, manifests, and weights need versioned, KMS-encrypted storage. POSIX filesystems lack atomicity and versioning at scale.

**Options**  
1. **AWS S3** (or GCP GCS): managed, KMS, versioning, lifecycle policies.  
2. **MinIO** (self-hosted S3 API): on-prem control, encryption, lower cost.  
3. **NFS + Git-LFS**: simple, but no KMS, weak concurrency, manual retention.

**Trade-offs**  
- S3: Mature, compliant, but vendor lock-in; egress costs.  
- MinIO: Open-source, deployable on-prem; team must manage HA/backups.  
- NFS: Familiar; scales poorly; no object lifecycle (DR-03: 1/3/7 year retention).

**Decision**  
**MinIO** for on-prem deployments, **S3** for cloud. Both expose S3 API (portability). Enable KMS encryption (NFR-S-01), versioning, and lifecycle rules (DR-03). Manifest checksums in DB index artifacts.

**Revisit Criteria**  
If data >100TB or multi-region replication needed, consolidate on cloud S3.

**Links**  
C4: Object Store, all processing containers | Stories: provenance, PHI de-ID | FR-004, NFR-S-01, DR-03

---

## ADR-004: PostgreSQL for Metadata with Audit Triggers

**Problem/Context**  
[SRS §4.8 FR-071, §5 NFR-O-01, Stories: audit logs, access logs]  
Every case, run, label edit, and export must be logged with timestamp, user, action. NoSQL lacks ACID and trigger support for audit trails.

**Options**  
1. **PostgreSQL** + audit triggers: ACID, RBAC, pgaudit extension.  
2. **MongoDB** + change streams: flexible schema, but eventual consistency.  
3. **Dedicated SIEM** (Splunk): powerful analytics, but high cost, delayed indexing.

**Trade-offs**  
- PostgreSQL: Strong consistency, native triggers, row-level security; schema migrations.  
- MongoDB: Fast writes, no joins; audit trail reconstruction complex.  
- SIEM: Best observability; overkill for structured metadata; 5+ sec latency.

**Decision**  
**PostgreSQL** with `pgaudit` and custom triggers for INSERT/UPDATE/DELETE on cases, runs, labels. Export audit table to SIEM weekly (NFR-O-01: 30-day retention). Row-level security enforces RBAC (FR-070).

**Revisit Criteria**  
If audit queries slow (>2s p95), partition audit table or add TimescaleDB extension.

**Links**  
C4: Metadata DB, API Gateway | Stories: audit logs, SSO+RBAC | FR-070, FR-071, NFR-O-01

---

## ADR-005: React SPA with TanStack Query for Review UI

**Problem/Context**  
[SRS §3.1 U-01..U-03, §5 NFR-U-01, Stories: overlay visualization, mask edits, <100ms latency]  
Reviewer must pan/zoom overlays, edit masks, and see diffs with sub-100ms tile latency. Server-side rendering adds roundtrip overhead.

**Options**  
1. **React SPA** + TanStack Query: client-side caching, optimistic updates.  
2. **Next.js SSR**: SEO-friendly, but tile fetch still client-side; complexity.  
3. **Dash/Plotly**: Python-native, but WebSocket lag for large masks.

**Trade-offs**  
- React SPA: Fast interactions, tile prefetch; no SSR (SEO irrelevant for internal tool).  
- Next.js: Unified stack; SSR unused; hydration overhead.  
- Dash: Minimal JS; Python callbacks add 50–200ms latency (violates NFR-U-01).

**Decision**  
**React SPA** with TanStack Query for tile caching. API Gateway serves 512×512 PNG tiles; UI prefetches adjacent tiles. WebGL overlay for mask blending. Optimistic UI for edits (FR-050 diff/rollback).

**Revisit Criteria**  
If tile latency >100ms p95, add CDN or lazy-load lower-res previews.

**Links**  
C4: Web UI, API Gateway | Stories: overlay viz, mask edits | U-01, U-02, NFR-U-01