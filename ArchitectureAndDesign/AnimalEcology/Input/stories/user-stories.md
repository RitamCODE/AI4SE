# User Stories

## Field Technician

- **US-01 — [Edge setup checklist]**  
  As a **Field Technician**, I want a guided setup checklist for camera traps and local network, so that I can install devices correctly the first time.

- **US-02 — [Health at a glance]**  
  As a **Field Technician**, I want a simple status page (CPU, disk, queue depth, last upload), so that I can quickly verify each unit is healthy.

- **US-03 — [Offline-first operation]**  
  As a **Field Technician**, I want the edge device to keep capturing and queuing data when the network is down, so that no evidence is lost during outages.

- **US-04 — [Ring buffer control]**  
  As a **Field Technician**, I want to set a storage cap and retention policy on the device, so that limited storage is used efficiently.

- **US-05 — [Adaptive capture rate]**  
  As a **Field Technician**, I want the device to automatically reduce capture rate when CPU or disk is high, so that the device stays responsive.

- **US-06 — [Remote update]**  
  As a **Field Technician**, I want to safely update edge software/firmware over the air with rollback, so that I can fix issues without visiting the site.

- **US-07 — [Log export]**  
  As a **Field Technician**, I want to export recent logs and metrics to a USB stick when offline, so that I can diagnose issues in the field.

---

## Drone Operator

- **US-08 — [Mission approval]**  
  As a **Drone Operator**, I want to review and approve auto-generated missions, so that I remain responsible for flight safety.

- **US-09 — [Auto-launch policy]**  
  As a **Drone Operator**, I want to define auto-launch rules (daylight, wind < threshold, battery ≥ threshold), so that safe missions can start without delays.

- **US-10 — [AOI geofence]**  
  As a **Drone Operator**, I want missions to be automatically geofenced to AOI polygons with buffers, so that the drone never flies outside approved areas.

- **US-11 — [Pre-flight checks]**  
  As a **Drone Operator**, I want automated pre-flight checks (GNSS lock, battery, link quality), so that risks are caught before takeoff.

- **US-12 — [Pause/abort & RTH]**  
  As a **Drone Operator**, I want to pause or abort a mission with a single action and trigger Return-to-Home, so that I can handle unexpected hazards.

- **US-13 — [Battery-aware planning]**  
  As a **Drone Operator**, I want missions split into segments when one battery is insufficient, so that the drone never risks a forced landing.

---

## Wildlife Scientist / Researcher

- **US-14 — [Define AOIs]**  
  As a **Wildlife Scientist**, I want to draw and version AOI polygons with metadata (altitude, overlap), so that surveys match research plans.

- **US-15 — [Species & thresholds]**  
  As a **Wildlife Scientist**, I want to set target species and detection confidence thresholds per AOI, so that only relevant events trigger missions.

- **US-16 — [ROI masking]**  
  As a **Wildlife Scientist**, I want to mask background regions (e.g., roads), so that false positives are reduced.

- **US-17 — [Event digest]**  
  As a **Wildlife Scientist**, I want a daily/weekly digest of validated events with exemplars, so that I can track activity without sifting through noise.

- **US-18 — [Label feedback]**  
  As a **Wildlife Scientist**, I want to mark detections as correct/incorrect and add species labels, so that model training improves over time.

- **US-19 — [Media review]**  
  As a **Wildlife Scientist**, I want to review drone-captured media aligned to AOI stripes and timestamps, so that I can assess coverage quality.

---

## Data / ML Engineer

- **US-20 — [Model versioning]**  
  As a **Data/ML Engineer**, I want to pin, roll out, and roll back on-device model versions, so that I can safely test improvements on subsets of devices.

- **US-21 — [Dataset curation]**  
  As a **Data/ML Engineer**, I want events and media auto-transferred to OSC with checksums and metadata, so that curated datasets are reproducible.

- **US-22 — [Tapis job templates]**  
  As a **Data/ML Engineer**, I want parameterized Tapis job templates (training vs inference), so that I can trigger consistent runs from new data arrivals.

- **US-23 — [Drift monitoring]**  
  As a **Data/ML Engineer**, I want precision/recall proxies and confidence histograms by AOI over time, so that I can detect model drift early.

- **US-24 — [Resource-aware models]**  
  As a **Data/ML Engineer**, I want the option to switch to a lighter model when compute/power is constrained, so that edge scoring stays within SLA.

- **US-25 — [Inference traceability]**  
  As a **Data/ML Engineer**, I want every detection to record model ID, config, and hash, so that I can trace how results were produced.

---

## DevOps / Platform Admin

- **US-26 — [RBAC & least privilege]**  
  As a **Platform Admin**, I want role-based access with least privilege and audit logs, so that sensitive actions are controlled and traceable.

- **US-27 — [Secrets management]**  
  As a **Platform Admin**, I want Tapis tokens and credentials stored in a secure vault with rotation, so that access remains safe over time.

- **US-28 — [Observability]**  
  As a **Platform Admin**, I want structured logs, metrics, and alerts (latency, queue depth, transfer failures), so that incidents can be detected and resolved quickly.

- **US-29 — [Config as code]**  
  As a **Platform Admin**, I want all device and pipeline configs versioned and deployable via CI/CD, so that changes are consistent and auditable.

- **US-30 — [Multi-tenant separation]**  
  As a **Platform Admin**, I want environments and storage segregated by project or site, so that data and policies don’t leak across teams.

---

## Program Manager / Principal Investigator

- **US-31 — [Operational KPIs]**  
  As a **Program Manager**, I want a dashboard of key metrics (events, missions, time-to-mission, transfer success), so that I can measure the value of automation.

- **US-32 — [Cost & resource view]**  
  As a **Program Manager**, I want to see storage, bandwidth, and compute usage trends, so that I can plan budgets and capacity.

---

## Compliance & Safety Officer

- **US-33 — [Flight policy compliance]**  
  As a **Compliance Officer**, I want missions to require explicit operator acknowledgment of local rules, so that operations remain lawful.

- **US-34 — [Privacy controls]**  
  As a **Compliance Officer**, I want optional blurring of humans and sensitive locations, so that captured media respects privacy.

- **US-35 — [Retention & purge]**  
  As a **Compliance Officer**, I want retention policies and auditable purges, so that data management follows institutional and legal requirements.

---

## Cross-Cutting / System Enablement

- **US-36 — [Event coalescing]**  
  As a **System**, I want to coalesce near-duplicate detections into a single event, so that alerts and transfers are not spammy.

- **US-37 — [Resumable transfers]**  
  As a **System**, I want chunked, checksum-verified, resumable uploads to OSC, so that intermittent networks don’t cause data loss.

- **US-38 — [Latency telemetry]**  
  As a **System**, I want to record timestamps for detection, mission ready, takeoff, capture, transfer, and job start, so that end-to-end latency can be improved.

- **US-39 — [Battery & weather gating]**  
  As a **System**, I want missions blocked when battery/forecast risk is high, so that safety is prioritized automatically.

- **US-40 — [Graceful degradation]**  
  As a **System**, I want to switch to lower frame rates or lighter models under load, so that the pipeline continues to function within constraints.

---

### Notes & Assumptions
- AOIs, thresholds, and policies can vary by site and are versioned.  
- Edge devices run Linux on Raspberry Pi–class hardware without GPU.  
- Parrot Anafi or compatible drones are available; operator remains in the loop unless auto-launch policies are met.  
- Tapis/OSC credentials and storage endpoints are provisioned in advance.

</details>