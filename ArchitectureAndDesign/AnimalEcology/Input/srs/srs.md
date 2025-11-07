# Software Requirements Specification (SRS)  
**Project:** Edge-to-Cloud Wildlife Intrusion Response (Camera-Trap → Drone → OSC Pipeline)

---

## 1. Document Control

| Field | Value |
|---|---|
| **Title** | Edge-to-Cloud Wildlife Intrusion Response System |
| **Version** | 0.1 (Draft) |
| **Date** | October 21, 2025 |
| **Author** | Project Team (replace with names/roles) |
| **Stakeholders** | Field Operations, Wildlife Scientists, Drone Operators, Data/ML Team, DevOps/Platform Team |

---

## 2. Introduction

### 2.1 Purpose
This document specifies the requirements for a system that detects animal intrusions via camera traps, scores images on-device, and—when warranted—automatically plans and executes drone reconnaissance over polygon Areas of Interest (AOIs). The system auto-transfers captured media to the Ohio Supercomputer Center (OSC) via Tapis to trigger training/inference workflows, with the overarching goal of reducing manual intervention across detection, data collection, transfer, and job orchestration.

### 2.2 Scope
- **In scope:** On-device detection and scoring; event filtering; AOI-constrained flight planning; automated drone data collection; reliable data transfer to OSC; Tapis-based job submission; monitoring; fault-tolerant buffering; role-based management.
- **Out of scope (initial release):** Long-range BVLOS flight approvals; advanced multi-drone swarming; automated airspace deconfliction beyond local geofences; model training algorithm design (covered by downstream OSC jobs).

### 2.3 Definitions, Acronyms, and Abbreviations
- **AOI:** Area of Interest (polygon defining survey bounds).  
- **Camera trap:** Edge device (e.g., Raspberry Pi) capturing stills/video on triggers.  
- **On-device scoring:** Running the object detector locally to assign confidence to detections.  
- **OSC:** Ohio Supercomputer Center (cloud/HPC target).  
- **Tapis:** API platform used to submit and orchestrate jobs on OSC.  
- **RTH:** Return-to-Home (drone failsafe action).  
- **UAV / Drone:** Parrot Anafi (or compatible) aircraft used for follow-up data collection.  
- **Event:** A scored detection that passes filters and can trigger planning/collection.  
- **GeoJSON/KML:** Formats for geospatial polygons/flight plans.

---

## 3. Overall Description

### 3.1 Product Perspective
The system spans **edge → near-edge → cloud**:
- **Edge (camera trap):** Capture, local scoring, deduplication, event filtering, ring-buffered storage, adaptive capture rate under resource constraints.
- **Drone layer:** AOI-constrained flight plan generation (e.g., lawnmower pattern), safety checks (geofence, battery, wind), autonomous media collection.
- **Cloud/HPC (OSC via Tapis):** Reliable, resumable transfer; staging; job submission (training/inference); status feedback to edge.

**High-level flow (textual):**  
`Camera trap → On-device scoring → Event filter/coalesce → AOI flight plan → Drone sortie → Media capture → Local batch → Tapis transfer → OSC job start → Results + metrics → Monitoring/alerts`

### 3.2 User Classes and Characteristics
- **Field Technician:** Installs and maintains camera traps and drones; limited time on site; needs robust defaults and simple UIs.
- **Drone Operator (Part 107/authorized):** Reviews/approves missions; monitors safety; may take manual control.
- **Wildlife Scientist/Researcher:** Defines AOIs, species of interest, alert thresholds; reviews media and analytics.
- **Data/ML Engineer:** Manages models, pipelines, and job parameters; validates training/inference runs on OSC.
- **DevOps/Platform Admin:** Oversees deployments, keys/credentials, observability, and security controls.

### 3.3 Assumptions and Dependencies
- **Assumptions:**  
  - Intermittent/limited bandwidth and power at edge; time synchronization available (NTP/GPS).  
  - Weather and regulatory constraints may prohibit immediate flight.  
  - AOIs are provided/approved by researchers and are lawful to overfly.  
- **Dependencies:**  
  - Parrot Anafi SDKs/APIs (or abstraction layer).  
  - Tapis credentials and OSC project allocations.  
  - MQTT/HTTPS connectivity (store-and-forward when offline).  
  - Supported file formats: images (JPEG/PNG), video (MP4), plans (JSON/KML), metadata (JSON/GeoJSON).

---

## 4. Functional Requirements

> **Notation:** Requirements are uniquely identified (FR-###) and grouped by capability. **Priority** uses MoSCoW (M=Must, S=Should, C=Could, W=Won’t for now).

### A. Detection & Scoring
- **FR-001 (M):** The camera trap SHALL capture frames on motion or schedule and store raw media with timestamp, GPS (if available), and device ID metadata.
- **FR-002 (M):** The edge device SHALL run an on-device object detector and assign detection confidences per frame.
- **FR-003 (M):** The system SHALL support configurable class filters (e.g., species of interest) and confidence thresholds per AOI.
- **FR-004 (S):** The system SHOULD support ROI masks to ignore static background regions and reduce false positives.
- **FR-005 (M):** The edge SHALL deduplicate near-identical frames using perceptual hashing or temporal coalescing to avoid redundant events.

### B. Event Filtering & Alerting
- **FR-006 (M):** The edge SHALL create an **Event** when detections meet AOI and threshold criteria and rate limits (cool-down windows).
- **FR-007 (S):** The system SHOULD coalesce multiple detections within a window into a single Event with summary stats (count, duration, exemplars).
- **FR-008 (S):** The system SHOULD notify designated roles (tech/operator/scientist) when an Event is created, with confidence and exemplar media.

### C. AOI & Flight Planning
- **FR-009 (M):** The system SHALL store AOIs as polygons (GeoJSON) with metadata (owner, effective dates, flight altitude, overlap, photo/video mode).
- **FR-010 (M):** Given an Event linked to an AOI, the system SHALL generate a **bounded** lawnmower/sweep flight plan that stays within the AOI and respects altitude/overlap constraints.
- **FR-011 (M):** The planner SHALL estimate sortie duration and battery usage; if infeasible, it SHALL split into segments or queue the mission.
- **FR-012 (S):** The planner SHOULD avoid no-fly sub-polygons and apply wind-aware leg orientation when data is available.
- **FR-013 (M):** The plan SHALL be exportable to the drone in a supported format (e.g., JSON/KML/SDK mission).

### D. Drone Operations & Safety
- **FR-014 (M):** The system SHALL require operator approval or policy-based auto-launch criteria (e.g., daylight, wind < X, visibility OK).
- **FR-015 (M):** The drone SHALL enforce geofences (AOI bounds + buffer), RTH on low battery/link loss, and log all failsafe triggers.
- **FR-016 (M):** The drone SHALL capture media according to plan (photo interval or video), attaching geotags and timestamps.
- **FR-017 (S):** The operator SHOULD be able to pause, resume, or abort missions; the system MUST record the final mission state.

### E. Data Management & Transfer
- **FR-018 (M):** The edge SHALL maintain a ring buffer with **configurable retention**; oldest items are purged when storage thresholds are reached.
- **FR-019 (M):** The edge SHALL batch files and compute checksums; transfers MUST be resumable (checkpointed) and integrity-verified end-to-end.
- **FR-020 (M):** The system SHALL adapt transfer concurrency and batch size to available bandwidth and power (throttling/backoff).
- **FR-021 (M):** On successful receipt at OSC, the edge SHALL safely mark batches as delivered and eligible for purge according to policy.
- **FR-022 (S):** The system SHOULD encrypt data at rest on edge (if HW permits) and MUST encrypt data in transit.

### F. Tapis / OSC Job Orchestration
- **FR-023 (M):** Upon completed transfer, the system SHALL create a Tapis job with references to the delivered dataset, model/config IDs, and runtime parameters.
- **FR-024 (M):** The system SHALL track job states (queued, running, succeeded, failed) and expose them to users and logs.
- **FR-025 (S):** The system SHOULD support job templates per AOI/species (e.g., inference vs. training) and parameterized runs (e.g., thresholds).
- **FR-026 (M):** Failures in job submission or execution SHALL trigger retries with exponential backoff and operator notifications.

### G. Configuration & Administration
- **FR-027 (M):** The system SHALL provide role-based access control (RBAC) for defining AOIs, thresholds, and flight policies.
- **FR-028 (M):** All configurations SHALL be versioned and auditable (who changed what, when).
- **FR-029 (S):** The system SHOULD offer a CLI/API and a minimal web UI for status, configs, and logs.

### H. Monitoring, Logging, and Telemetry
- **FR-030 (M):** The system SHALL log key events (capture, detection, filter, plan, launch, media captured, transfers, Tapis jobs).
- **FR-031 (S):** The edge SHOULD expose a health endpoint (CPU, disk, queue depth, battery state for drone dock if available).
- **FR-032 (M):** The system SHALL generate metrics to quantify end-to-end latency (detection → drone launch → data arrival → job start).

### I. Resilience & Offline Operation
- **FR-033 (M):** The edge SHALL queue Events and batches offline and forward them when connectivity resumes.
- **FR-034 (M):** The system SHALL degrade capture rate automatically when CPU/storage thresholds are exceeded (backpressure).
- **FR-035 (S):** The system SHOULD allow manual export via portable media when network is unavailable for > configurable period.

### J. Security & Compliance (functional behaviors)
- **FR-036 (M):** The system SHALL store credentials (e.g., Tapis tokens) using an OS keyring or encrypted vault; rotation MUST be supported.
- **FR-037 (M):** The system SHALL enforce AOI-based flight boundaries and record operator acknowledgments required by local policy.
- **FR-038 (S):** The system SHOULD support per-dataset access policies for researchers (read-only vs. manage jobs).

---

## 5. Non-Functional Requirements (NFRs)

### 5.1 Performance
- **NFR-P1 (M):** On-device scoring median time ≤ **2 s/frame** on target Raspberry Pi class hardware for configured model.
- **NFR-P2 (M):** Median end-to-end latency (Event created → drone mission ready) ≤ **5 minutes** when pre-approved auto-launch policy applies and weather is acceptable.
- **NFR-P3 (S):** Transfer pipeline to OSC sustains ≥ **1 Mbps** under constrained links with adaptive throttling; zero data corruption (checksum verified).

### 5.2 Reliability & Availability
- **NFR-R1 (M):** ≥ **99%** successful, integrity-verified delivery of media within **24 hours** under intermittent connectivity.
- **NFR-R2 (M):** No single failure at edge (process crash, temporary power loss) results in data loss beyond ring-buffer policy.
- **NFR-R3 (S):** Drone mission completion success rate ≥ **95%** under nominal conditions (battery, GPS lock, link OK).

### 5.3 Scalability
- **NFR-S1 (M):** Support **≥100** camera traps and **≥20** concurrent AOIs; queue and schedule drone sorties without starvation.
- **NFR-S2 (S):** Support **multi-region** deployments, with separated Tapis/OSC tenants by configuration.

### 5.4 Security & Privacy
- **NFR-SEC1 (M):** TLS for all control and data transfers; AES-based encryption at rest where hardware permits.
- **NFR-SEC2 (M):** RBAC with least privilege; audit logs immutable for **≥180 days**.
- **NFR-SEC3 (S):** PII-safe defaults; allow blurring of human subjects if accidentally captured.

### 5.5 Maintainability & Operability
- **NFR-M1 (M):** Configuration-as-code with validation; changes deployable via CI/CD to edge.
- **NFR-M2 (M):** Observability: structured logs, metrics, alerting thresholds; log search by device/AOI/Event ID.
- **NFR-M3 (S):** Mean time to diagnose critical incident ≤ **30 min** with standard runbooks.

### 5.6 Portability & Compatibility
- **NFR-PC1 (M):** Edge agents run on 64-bit Linux (Debian/Raspberry Pi OS) with no GPU required.
- **NFR-PC2 (S):** Drone abstraction layer supports at least **Parrot Anafi**; future connectors pluggable.

### 5.7 Usability
- **NFR-U1 (S):** Minimal UI workflows complete in ≤ **5 clicks** (approve mission, view job status, adjust threshold).
- **NFR-U2 (S):** Operator feedback (warnings, flight estimates) presented in plain language with actionable next steps.

---

## 6. Validation Criteria

| Requirement Group | Validation Method | Success Criteria |
|---|---|---|
| Detection & Scoring (FR-001–005) | Unit + edge integration tests with labeled test set | ≥ target precision/recall on device; ≤ 2 s/frame median; dedup removes ≥80% near-duplicates in test batch |
| Event Filtering (FR-006–008) | Simulation with synthetic bursts | Coalescing reduces event spam ≥60% without missing true positives in test scenarios |
| AOI & Planning (FR-009–013) | Planner unit tests + geofence tests | All generated waypoints lie within AOI; plan duration/battery estimates within ±10% of flight logs |
| Drone Ops & Safety (FR-014–017) | Field tests under nominal conditions | RTH triggers at threshold; geofence violations prevented; pause/resume/abort logged |
| Data Transfer (FR-018–022) | Network impairment tests | Resume after drop; checksums match; adaptive throttling keeps CPU < 80% and storage > 10% free |
| Tapis/OSC Jobs (FR-023–026) | Staging & sandbox runs | Jobs created with correct inputs; status reflected; failed jobs retried N times with backoff |
| Config & RBAC (FR-027–029) | Access tests, audit review | Unauthorized changes blocked; config history complete; API/CLI consistent |
| Monitoring & Telemetry (FR-030–032) | Log/metric inspection | End-to-end latency metric emitted; trace spans correlate Event → Job |
| Resilience/Offline (FR-033–035) | Power/cable pull tests | No data loss beyond policy; automatic forwarding on reconnect |
| Security/Compliance (FR-036–038) | Pen test + config review | Secrets encrypted; token rotation works; AOI flight boundary enforced |

**Acceptance Test Examples**
- **AT-1:** Simulate an intrusion event; verify plan generation ≤ 60 s and operator approval flow; verify drone captures geotagged photos across AOI stripes.  
- **AT-2:** Impair link to 100 kbps; verify batched, resumable transfer and Tapis job creation within 2 hours.  
- **AT-3:** Force low battery in flight; verify RTH and complete logs; ensure incomplete media remains queued and later delivered.

---

## 7. Appendix

### 7.1 Glossary
- **Backpressure:** Automatic reduction of capture/transfer rate when resources are constrained.  
- **Coalescing:** Combining temporally adjacent detections into a single representative Event.  
- **Ring buffer:** Fixed-size storage that overwrites oldest items when full.

### 7.2 Constraints (Design Guidance)
- **Limited storage & power:** Prefer lightweight models; enforce retention policies; compress media.  
- **Intermittent networks:** Use resumable, checksum-verified uploads; prefer bulk/batched transfers to amortize overhead.  
- **Operational safety:** Always bound plans to AOIs; require operator approval unless policy allows auto-launch.

### 7.3 Initial Configuration Checklist (Informative)
- Define AOIs (GeoJSON) and flight policies (altitude, overlap, media mode).  
- Set detection classes and thresholds per AOI/species.  
- Provision Tapis credentials and OSC project paths.  
- Configure ring-buffer size, transfer windows, and alert recipients.  
- Validate a dry-run (simulated Event → plan → mock transfer → test job).

---

## 8. Requirement Summary Tables (Optional)

### 8.1 Functional Requirements by Priority
| Priority | Count | IDs (range) |
|---|---|---|
| Must (M) | 26 | FR-001–004,006,009–011,013–016,018–021,023–024,026–028,030,032–034,036–037 |
| Should (S) | 12 | FR-005,007–008,012,017,022,025,029,031,035,038 |
| Could (C) | 0 | — |

### 8.2 Key Performance Targets
| Metric | Target |
|---|---|
| On-device scoring | ≤ 2 s/frame (median) |
| Event→Mission ready | ≤ 5 min (auto-launch policy met) |
| Delivery integrity | 100% checksum verified |
| Drone mission success | ≥ 95% under nominal conditions |

---

**End of Document**

</details>