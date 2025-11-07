
---

### 3. `03-adrs.md`
```markdown
# Architecture Decision Records (ADRs)

## ADR-001: Monolithic vs. Microservices
- **Problem:** How to structure the pipeline for maintainability and scalability? (SRS 2.2, 2.3, 4.1–4.9)
- **Options:**
  1. **Monolithic pipeline** (single container, shared memory).
  2. **Microservices** (per-stage containers, gRPC/REST).
- **Trade-offs:**
  - Monolithic: Simpler deployment, but harder to scale/isolate stages.
  - Microservices: Overhead for inter-service comms, but aligns with NFR-M-02 (containerized) and NFR-R-01 (retry/idempotency).
- **Decision:** Microservices with gRPC for internal comms (low latency) and REST for external APIs.
- **Revisit:** If gRPC overhead exceeds 10% of stage runtime.

---

## ADR-002: Provenance Storage
- **Problem:** How to store immutable run manifests for reproducibility? (SRS 4.1, FR-003, FR-004)
- **Options:**
  1. **Blockchain-ledger** (tamper-proof, but complex).
  2. **Versioned object storage + SQL index** (simple, auditable).
- **Trade-offs:**
  - Blockchain: Overkill for internal use; violates NFR-M-02 (portability).
  - Versioned storage: Meets FR-004 (immutable artifacts) and NFR-S-01 (encryption).
- **Decision:** Versioned S3 + Postgres index.
- **Revisit:** If audit failures exceed 0.1% of runs.

---

## ADR-003: Human-in-the-Loop UI
- **Problem:** How to enable real-time mask editing with low latency? (SRS 4.6, NFR-U-01)
- **Options:**
  1. **Server-rendered PNGs** (simple, but high latency).
  2. **WebGL + tile streaming** (complex, but meets <100ms requirement).
- **Trade-offs:**
  - Server-rendered: Fails NFR-U-01 (<100ms).
  - WebGL: Requires frontend investment, but aligns with U-02 (review tool).
- **Decision:** WebGL + tile streaming with CDN caching.
- **Revisit:** If tile latency exceeds 150ms p95.

---

## ADR-004: Model Deployment
- **Problem:** How to deploy models safely for canary testing? (SRS 4.7, FR-061)
- **Options:**
  1. **Blue-green deployment** (zero downtime, but no gradual rollout).
  2. **Shadow deployment + metric gates** (gradual, but doubles inference cost).
- **Trade-offs:**
  - Blue-green: Fails FR-061 (canary).
  - Shadow: Aligns with NFR-R-01 (retry/idempotency) and FR-061 (promotion gates).
- **Decision:** Shadow deployment with 10% traffic, metric gates for promotion.
- **Revisit:** If shadow cost exceeds 20% of inference budget.

---

## ADR-005: PHI Handling
- **Problem:** How to ensure PHI is never exposed in logs/exports? (SRS 2.4, FR-072)
- **Options:**
  1. **Field-level encryption** (secure, but complex queries).
  2. **Redaction filters + DLP scans** (simpler, but requires validation).
- **Trade-offs:**
  - Encryption: Violates NFR-P-01 (throughput).
  - Redaction: Meets NFR-S-02 (no raw PHI in logs) and FR-080 (de-identified exports).
- **Decision:** Redaction filters + automated DLP scans on logs/exports.
- **Revisit:** If PHI leak detected in audit.
