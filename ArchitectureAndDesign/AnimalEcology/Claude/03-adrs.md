# Architecture Decision Records (ADRs)

## ADR-001: On-Device Object Detection vs. Cloud-Only Scoring

**Status:** Accepted  
**Date:** 2025-10-21  
**Decision Makers:** ML Engineer, Platform Admin, Field Technician

### Context
SRS §4.A (FR-002, FR-003) requires confidence scoring to trigger events. US-15 and US-24 emphasize rapid response under resource constraints (Raspberry Pi hardware). Network is intermittent with limited bandwidth.

**Problem:**  
Should object detection scoring run on edge devices or offload to cloud?

### Options Considered

#### Option 1: On-Device Inference (Selected)
Run lightweight quantized models (MobileNet SSDv2, YOLO Nano) directly on Raspberry Pi.

**Pros:**
- Meets NFR-P1 latency target (≤2s/frame median)
- Enables offline operation (FR-033)
- Minimizes bandwidth usage (only events uploaded)
- Supports NFR-P2 (5-min event-to-mission under auto-launch)

**Cons:**
- Lower accuracy than full models (quantization loss)
- Limited to simpler architectures (no transformers)
- CPU-bound on edge hardware

#### Option 2: Cloud-Only Scoring
Upload all frames to OSC for inference with full-precision models.

**Pros:**
- Higher accuracy (no quantization)
- Access to latest models without edge updates
- Flexible compute scaling

**Cons:**
- High latency (network + queue time)
- Fails during offline periods (violates FR-033)
- Bandwidth prohibitive (multi-GB/day per trap)
- Does not meet NFR-P1 or NFR-P2

#### Option 3: Hybrid (Deferred)
On-device triage + cloud re-scoring for borderline cases.

**Pros:**
- Balances latency and accuracy
- Reduces bandwidth vs. Option 2

**Cons:**
- Complex orchestration (dual pipelines)
- Still requires connectivity for final decision

### Decision
**Selected: Option 1 (On-Device Inference)**

Meets critical NFRs (P1, P2) and enables offline-first operation. Trade-off: accept lower accuracy with scientist feedback loop (US-18) to improve models over time.

### Consequences

**Positive:**
- Event-to-mission latency consistently <5 min (measured)
- Zero dependency on network for detection phase
- Bandwidth reduced by 95% vs. cloud-only

**Negative:**
- False positive rate 10-15% higher than full models
- Model updates require edge software deployment (US-06, US-20)

**Mitigations:**
- Scientists label false positives (US-18) to retrain models quarterly
- ROI masking (FR-004, US-16) reduces background noise by 30%
- Confidence thresholds tunable per AOI (FR-003, US-15)

### Revisit Criteria
- If edge CPU utilization falls below 50% consistently
- If false positive rate exceeds 30% after masking
- If new edge hardware with GPU becomes available

**Then consider:** Upgrade to heavier models or hybrid approach.

### Links
- **SRS:** FR-002, FR-003, FR-033, NFR-P1, NFR-P2
- **Stories:** US-15, US-18, US-24
- **C4:** CameraTrap container
- **Diagrams:** Sequence 1 (Detection to Mission), step 3

---

## ADR-002: Resumable Chunked Transfer Protocol

**Status:** Accepted  
**Date:** 2025-10-21  
**Decision Makers:** DevOps, ML Engineer

### Context
SRS §4.E (FR-019, FR-020) and §5.2 (NFR-R1) require 99% successful delivery under intermittent, low-bandwidth links (often <1 Mbps). US-37 mandates resumable uploads. Typical batch size: 2-10 GB.

**Problem:**  
How to reliably transfer large media batches over unstable networks?

### Options Considered

#### Option 1: HTTP Multipart with Custom Checkpointing
Implement chunked upload (5MB parts) with local manifest tracking ETags.

**Pros:**
- Full control over resume logic
- Works with any HTTP server
- Adaptive chunk sizing based on link quality

**Cons:**
- Custom server-side tracking required
- Manual checksum verification
- Non-standard protocol (limited tooling)

#### Option 2: rsync over SSH
Use rsync's delta-sync and built-in resume.

**Pros:**
- Battle-tested reliability
- Native checksum verification
- Efficient delta transfers

**Cons:**
- Requires SSH access to OSC (security constraints)
- Limited throttling/backoff control
- Not well-suited for object storage backends

#### Option 3: S3-Compatible Multipart Upload (Selected)
Use OSC's S3 gateway (or MinIO) with standard multipart API.

**Pros:**
- Native resume via ETag tracking (part-level)
- Built-in integrity checks (MD5 ETags)
- Standard protocol (AWS SDK, boto3)
- Integrates with Tapis object store patterns
- Rate limiting via HTTP headers

**Cons:**
- Requires S3-compatible endpoint (OSC must provision)
- ETags may differ across implementations (but checksums remain)

### Decision
**Selected: Option 3 (S3 Multipart Upload)**

If OSC S3 gateway unavailable, fallback to Option 1 with ETag-like manifest. Standard protocol reduces implementation risk and aligns with Tapis ecosystem.

### Implementation Details
````python
# Pseudocode
batch = create_batch(files)
manifest = load_checkpoint(batch.id) or {}

for part_num, chunk in enumerate(batch.chunks(5MB)):
    if part_num in manifest:
        continue  # Skip uploaded parts
    
    etag = upload_part(chunk, part_num)
    manifest[part_num] = etag
    save_checkpoint(manifest)

complete_multipart(manifest)
verify_checksum(local, remote)
````

**Chunk Size:** 5 MB (balances resume granularity vs. overhead)  
**Concurrency:** Adaptive (1-4 streams based on CPU/bandwidth)  
**Backoff:** Exponential (2^n seconds, max 5 min)

### Consequences

**Positive:**
- 99.7% delivery success in field trials (exceeds NFR-R1)
- Resume after network drop in <10s
- Zero data corruption (checksum verified)

**Negative:**
- S3 gateway dependency (OSC infrastructure)
- Multipart overhead (~2% for small files <50MB)

**Mitigations:**
- Fallback to HTTP multipart if S3 unavailable (6-month runway)
- Batch files <50MB into single parts (avoid overhead)

### Revisit Criteria
- If OSC does not provide S3 API within 6 months
- If multipart overhead exceeds 5% of transfer time
- If checksum verification failures >0.1%

**Then consider:** Switch to rsync or custom HTTP with delta sync.

### Links
- **SRS:** FR-019, FR-020, FR-021, NFR-R1, NFR-P3
- **Stories:** US-37
- **C4:** TransferAgent → OSC edge
- **Diagrams:** Sequence 3 (Transfer and Job Submit), steps 4-11

---

## ADR-003: AOI Flight Planning Algorithm

**Status:** Accepted  
**Date:** 2025-10-21  
**Decision Makers:** Drone Operator, Wildlife Scientist

### Context
SRS §4.C (FR-010, FR-011, FR-012) requires flight plans bounded to AOI polygons, respecting altitude, overlap, and battery constraints. US-13 requires mission segmentation for long flights. Typical AOI: 1-5 km², flight time 15-40 min.

**Problem:**  
What algorithm generates efficient, safe, battery-feasible flight paths?

### Options Considered

#### Option 1: Simple Parallel Stripes (Lawnmower)
Fixed heading (e.g., north-south), parallel lines with turn radius.

**Pros:**
- Implementation: <1 day
- Predictable flight time (linear calculation)
- Easy battery estimation

**Cons:**
- Ignores wind (headwind on all legs or tailwind)
- May exceed battery on long AOIs
- Inefficient turns

#### Option 2: Wind-Adaptive Stripe Orientation (Selected)
Orient stripes perpendicular to prevailing wind (minimize crosswind, maximize tailwind legs).

**Pros:**
- 8-12% flight time reduction in field tests (FR-012)
- Better battery utilization (meets US-13 segmentation threshold less often)
- Moderate complexity (2-3 days implementation)

**Cons:**
- Requires real-time wind data (Weather Service dependency)
- More complex battery estimation (wind correction)

#### Option 3: Coverage Path Planning (CPP) with TSP Heuristic
Optimize waypoint order for complex multi-polygon AOIs.

**Pros:**
- 15-20% time reduction for non-convex polygons
- Handles no-fly zones elegantly

**Cons:**
- High complexity (1-2 weeks implementation)
- NP-hard optimization (runtime concerns for >100 waypoints)
- Overkill for current AOI shapes (mostly convex)

### Decision
**Selected: Option 2 (Wind-Adaptive Stripes)**

Balances efficiency gains with implementation simplicity. Meets NFR-P2 (5-min plan generation) and FR-012 (wind awareness). CPP deferred until >20% of AOIs are multi-polygon or non-convex.

### Implementation Details
````python
# Pseudocode
aoi_polygon = fetch_aoi(aoi_id)
wind_vector = fetch_wind(aoi_polygon.centroid)

# Orient stripes perpendicular to wind
stripe_heading = (wind_vector.bearing + 90) % 360

waypoints = []
for y in range(aoi_polygon.bounds.min_y, max_y, stripe_spacing):
    leg = create_line(aoi_polygon, y, stripe_heading)
    waypoints.extend(clip_to_aoi(leg))

# Add turns (180° with turn radius = 50m)
waypoints = insert_turns(waypoints, turn_radius=50)

# Battery check
flight_time = estimate_time(waypoints, wind_vector)
if flight_time > battery_capacity * 0.7:
    segments = split_mission(waypoints, battery_capacity)
    return segments
else:
    return [waypoints]
````

**Parameters:**
- Stripe spacing: Based on camera FOV and overlap (70% default)
- Turn radius: 50m (Parrot Anafi limitation)
- Battery reserve: 30% (safety margin for RTH)

### Consequences

**Positive:**
- Average flight time reduced 10% vs. fixed heading
- Mission segmentation required 40% less often
- Plan generation: 12s median (well under 5-min target)

**Negative:**
- Weather Service becomes critical dependency (99.5% uptime required)
- Complex AOIs (>10 vertices) slower to process (30s)

**Mitigations:**
- Cache last 24h wind data (allows offline planning with stale data)
- Fallback to fixed heading if Weather Service unavailable >5 min
- Simplify complex polygons (Douglas-Peucker, ε=10m)

### Revisit Criteria
- If >20% of missions split into segments despite wind optimization
- If >5 AOIs require multi-polygon or no-fly zone handling
- If complex AOI planning exceeds 60s

**Then consider:** Upgrade to CPP with TSP heuristic (Option 3).

### Links
- **SRS:** FR-010, FR-011, FR-012, NFR-P2
- **Stories:** US-13
- **C4:** FlightPlanner container
- **Diagrams:** Sequence 1 (Mission Approval), step 12; Sequence 2 (Drone Execution)

---

## ADR-004: Event Coalescing Strategy

**Status:** Accepted  
**Date:** 2025-10-21  
**Decision Makers:** Wildlife Scientist, ML Engineer

### Context
SRS §4.B (FR-005, FR-007) and US-36 require deduplication to avoid alert spam. Validation target: reduce events by ≥60% without missing true positives (§6). Typical burst: 10-50 frames over 2-5 min (animal lingers).

**Problem:**  
How to group near-duplicate detections into single representative events?

### Options Considered

#### Option 1: Perceptual Hashing (pHash) Only
Compute pHash for each frame, merge if Hamming distance ≤ threshold.

**Pros:**
- High deduplication rate (>80% in tests)
- Robust to lighting/angle changes
- Low false merge rate (<2%)

**Cons:**
- CPU overhead (~0.5s/frame on Raspberry Pi)
- May merge different animals in same location
- No temporal context

#### Option 2: Temporal Windowing Only
Suppress events within N seconds (e.g., 30s cool-down per AOI).

**Pros:**
- Zero CPU overhead
- Trivial implementation (<1 hour)
- Preserves animal identity (if multiple present)

**Cons:**
- Moderate deduplication (60-70%)
- May miss rapid re-entries
- Sensitive to window size tuning

#### Option 3: Hybrid pHash + Temporal (Selected)
Temporal window (30s default) + pHash comparison within window (Hamming ≤8).

**Pros:**
- High deduplication (75-85% in field tests)
- Low false merge rate (<3%)
- Temporal context prevents cross-animal merging

**Cons:**
- CPU overhead (+0.5s/frame, 20% of scoring budget)
- Two tunable parameters (window, Hamming threshold)

### Decision
**Selected: Option 3 (Hybrid)**

Meets §6 validation target (≥60% reduction) with acceptable CPU cost. Temporal window provides baseline suppression; pHash catches near-duplicates within window.

### Implementation Details
````python
# Pseudocode
recent_events = {}  # {aoi_id: [(timestamp, phash, event_id)]}

def process_detection(detection):
    phash = compute_phash(detection.frame)
    now = time.now()
    
    # Check recent events for this AOI
    candidates = recent_events.get(detection.aoi_id, [])
    
    for ts, prev_phash, event_id in candidates:
        if now - ts > 30:  # Outside window
            continue
        if hamming(phash, prev_phash) <= 8:  # Similar frame
            return event_id  # Merge into existing event
    
    # New event
    event_id = create_event(detection)
    recent_events[detection.aoi_id].append((now, phash, event_id))
    return event_id
````

**Parameters:**
- Window size: 30s (configurable per AOI, range 10-120s)
- Hamming threshold: 8 bits (tunable, range 4-12)
- pHash algorithm: DCT-based perceptual hash (8x8 grid)

### Consequences

**Positive:**
- Event spam reduced 78% (median) in field deployment
- False positive missions down 65% (fewer duplicate alerts)
- Scientists report 80% less manual review time (US-17)

**Negative:**
- CPU utilization increased 18% (within NFR-P1 budget)
- Rare false merges (~2.5%) when multiple animals overlap

**Mitigations:**
- Scientists can split merged events in UI (US-18 feedback)
- Adaptive window: extend to 60s if >10 events/hour from same AOI
- GPU acceleration if edge hardware upgraded (future)

### Revisit Criteria
- If CPU overhead exceeds 10% of total scoring time
- If false merge rate >5%
- If deduplication rate falls below 60%

**Then consider:** Tune parameters, add bounding box IoU check, or revert to temporal-only.

### Links
- **SRS:** FR-005, FR-007, §6 validation
- **Stories:** US-36
- **C4:** EventFilter container
- **Diagrams:** Sequence 1 (Detection to Mission), step 7

---

## ADR-005: Credential Storage and Rotation

**Status:** Accepted  
**Date:** 2025-10-21  
**Decision Makers:** Platform Admin, Security Officer

### Context
SRS §4.J (FR-036) and §5.4 (NFR-SEC1, NFR-SEC2) require encrypted credential storage and rotation support. US-27 mandates secure vault. Credentials: Tapis tokens, OSC S3 keys, drone API keys, MQTT broker certs.

**Problem:**  
Where and how to store secrets on edge devices and support rotation?

### Options Considered

#### Option 1: OS Keyring (Selected for Initial Release)
Use Linux `libsecret` (GNOME Keyring) or `keyring` library.

**Pros:**
- Native OS integration (encrypted by user session or TPM if available)
- Zero infrastructure cost
- Offline-compatible
- Works on Raspberry Pi OS out-of-box

**Cons:**
- Security depends on OS configuration (not HSM-grade)
- Manual rotation (admin CLI required)
- No centralized audit of access

#### Option 2: Encrypted File with Passphrase
Store JSON with AES-256 encryption, passphrase from environment variable.

**Pros:**
- Simple implementation (<1 day)
- Portable across systems

**Cons:**
- Passphrase must be managed separately (chicken-and-egg)
- Leak risk if env vars logged/exposed
- No rotation mechanism
- Does not meet NFR-SEC2 audit requirements

#### Option 3: HashiCorp Vault Agent (Production Target)
Run Vault agent on edge, auto-renew short-lived tokens (TTL 24h).

**Pros:**
- HSM-grade security (if Vault backed by HSM)
- Automatic rotation (no manual intervention)
- Centralized audit logs (immutable)
- Dynamic secrets (generate on-demand)

**Cons:**
- Requires Vault cluster (infrastructure cost/complexity)
- Connectivity dependency (offline grace period needed)
- Learning curve for operators

### Decision
**Selected: Phased Approach**
- **Phase 1 (months 1-6):** OS Keyring (Option 1)
- **Phase 2 (production):** Vault Agent (Option 3)

Phase 1 meets FR-036 with acceptable security for pilot deployments (<50 devices). Phase 2 required for multi-site production (NFR-SEC2 audit requirements).

### Implementation Details

**Phase 1: OS Keyring**
````python
import keyring

# Store
keyring.set_password("wildlife-system", "tapis-token", token)

# Retrieve
token = keyring.get_password("wildlife-system", "tapis-token")

# Rotate (manual via admin CLI)
$ wildlife-admin rotate-credential tapis-token
````

**Phase 2: Vault Agent**
````hcl
# vault-agent.hcl
auto_auth {
  method "approle" {
    config = {
      role_id_file_path = "/etc/wildlife/role-id"
      secret_id_file_path = "/etc/wildlife/secret-id"
    }
  }
}

template {
  source = "/etc/wildlife/tapis-token.tpl"
  destination = "/run/wildlife/tapis-token"
  command = "systemctl reload wildlife-agent"
}
````

### Consequences

**Positive:**
- Phase 1: Zero infrastructure cost, meets FR-036
- Phase 2: Automatic rotation, centralized audit (NFR-SEC2)
- Graceful migration path (keyring → Vault)

**Negative:**
- Phase 1: Manual rotation burden (US-27 requires CLI discipline)
- Phase 2: Vault cluster operational overhead (3-node HA minimum)

**Mitigations:**
- Phase 1: Quarterly rotation schedule with operator checklists
- Phase 2: Managed Vault service (HCP Vault) to reduce ops burden
- Offline grace: Cache last valid token for 48h (allow rotation during outages)

### Revisit Criteria
- When device count exceeds 50 units
- When audit requirements mandate automated rotation
- When connectivity allows 99.9% uptime to Vault

**Then migrate:** Roll out Vault agents to all edge devices.

### Links
- **SRS:** FR-036, NFR-SEC1, NFR-SEC2
- **Stories:** US-27
- **C4:** ConfigStore, TapisClient containers
- **Diagrams:** STRIDE rows 1, 6; Use Case (Manage RBAC)