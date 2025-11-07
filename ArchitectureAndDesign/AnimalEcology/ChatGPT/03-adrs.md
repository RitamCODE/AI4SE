
---

```markdown
# Architecture Decision Records (ADRs)

## ADR-001 — Edge-centric pipeline with cloud orchestration

- Context
  - Detection and triggering must work with intermittent connectivity and constrained edge devices.
  - SRS: FR-001–004, FR-006–013, FR-018; relevant SRS Sections 3–4.
  - Stories: US-01, US-02, US-03, US-16–US-19. 
- Options
  - A: Cloud-only detection; edge streams raw media.
  - B: Edge detection + buffering + policy; cloud for heavy jobs. (Chosen)
  - C: Regional aggregators between edge and cloud.
- Trade-offs
  - A: Simple; breaks offline requirements; high bandwidth.
  - B: Better resilience and bandwidth; more edge complexity.
  - C: Flexible; higher ops cost; not needed at current scale.
- Decision
  - Use EdgeCaptureScoring + EdgeEventPolicy + EdgeDataTransfer on-device; TapisOSCOrchestrator in cloud.
- Revisit
  - When device scale or hardware profile changes significantly.
- Links
  - C4: EdgeCaptureScoring, EdgeEventPolicy, EdgeDataTransfer.
  - Stories: US-01–US-03, US-16–US-19.
  - NFR: NFR-001 (on-device scoring, Assumed), NFR-003 (integrity).
  
---

## ADR-002 — Chunked, checksum-verified, resumable transfer

- Context
  - Need reliable uploads from unstable links.
  - SRS: FR-018–FR-021, FR-032.
  - Stories: US-24–US-28.
- Options
  - A: Single-shot file uploads.
  - B: Chunked uploads with per-chunk checksum and resume tokens. (Chosen)
  - C: Custom streaming protocol over gRPC.
- Trade-offs
  - A: Easiest; fragile; violates reliability FRs.
  - B: Robust; moderate implementation cost.
  - C: Powerful; harder integration with Tapis.
- Decision
  - EdgeDataTransfer uses chunked uploads, `batchId` idempotency, manifests.
- Revisit
  - If Tapis provides first-class multi-part APIs aligning with our needs.
- Links
  - C4: EdgeDataTransfer ↔ TapisOSC.
  - NFR: NFR-003 (Assumed: 100% verified chunks), NFR-002 (Assumed latency bounds).

---

## ADR-003 — Tapis/OSC as primary compute orchestrator

- Context
  - Need scalable training/inference without reinventing job scheduling.
  - SRS: FR-023–FR-026.
  - Stories: US-29–US-34.
- Options
  - A: Directly use Tapis jobs. (Chosen)
  - B: Custom scheduler and SSH-based submissions.
  - C: Hybrid multi-backend orchestrator.
- Trade-offs
  - A: Uses existing auth/quota; simpler integration.
  - B: More control; high maintenance and risk.
  - C: Flexible; unnecessary complexity now.
- Decision
  - All compute flows through TapisOSCOrchestrator using Tapis APIs only.
- Revisit
  - If non-Tapis backends or multi-cloud are added.
- Links
  - C4: TapisOSCOrchestrator ↔ TapisOSC.
  - NFR: NFR-006 (Assumed: ≥99% job submission success), NFR-007 (Assumed: state sync ≤60s).

---

## ADR-004 — Dedicated MissionPlannerDroneGateway for flight safety

- Context
  - Drone missions require strict gating, geofencing, and auditability.
  - SRS: FR-014–FR-017.
  - Stories: US-10–US-15.
- Options
  - A: Mission logic inside EdgeEventPolicy.
  - B: Dedicated MissionPlannerDroneGateway service. (Chosen)
  - C: Rely solely on vendor cloud for planning.
- Trade-offs
  - A: Tight coupling; hard to certify.
  - B: Clear boundary, safer, traceable.
  - C: Vendor lock-in; AOI semantics lost.
- Decision
  - All missions flow through MissionPlannerDroneGateway; it enforces rules and logs.
- Revisit
  - If fleet composition or regulations require different interfaces.
- Links
  - C4: EdgeEventPolicy ↔ MissionPlannerDroneGateway ↔ DroneFleet.
  - NFR: NFR-008 (Assumed: ≥99.9% missions respect constraints).

---

## ADR-005 — Central versioned configuration via ConfigAdminAPI

- Context
  - AOIs, thresholds, policies, and models must be consistent and auditable.
  - SRS: FR-027–FR-029, FR-030.
  - Stories: US-04–US-09.
- Options
  - A: Manual, per-device configs.
  - B: ConfigAdminAPI with RBAC, versions, signed configs. (Chosen)
  - C: Direct exposure of generic config store.
- Trade-offs
  - A: Error-prone, no traceability.
  - B: Clear ownership, audit, rollback.
  - C: Adds complexity; weaker domain semantics.
- Decision
  - ConfigAdminAPI as single config authority; edge pulls/pushes by `configVersion`.
- Revisit
  - If multi-tenant or large-scale deployments demand external config backbone.
- Links
  - C4: ConfigAdminAPI ↔ EdgeCaptureScoring/EdgeEventPolicy/MissionPlannerDroneGateway.
  - NFR: NFR-009 (Assumed: propagation ≤5min), NFR-010 (Assumed: all changes logged).
