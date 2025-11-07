# C4: Container

## Container Diagram - Edge-to-Cloud Wildlife System
```mermaid
C4Container
    title Container Diagram - Edge-to-Cloud Wildlife System

    Container_Boundary(edge, "Edge Layer") {
        Container(cameraTrap, "Camera Trap Agent", "Python/Linux", "Captures frames, runs on-device scoring, deduplicates")
        Container(eventFilter, "Event Filter Service", "Python", "Coalesces detections, applies thresholds, creates Events")
        Container(ringBuffer, "Ring Buffer Store", "Filesystem/SQLite", "Stores media with retention policy")
    }

    Container_Boundary(drone, "Drone Layer") {
        Container(flightPlanner, "Flight Planner", "Python", "Generates AOI-constrained lawnmower plans")
        Container(droneController, "Drone Controller", "Python/Parrot SDK", "Executes missions, enforces geofence, handles RTH")
    }

    Container_Boundary(cloud, "Cloud/Transfer Layer") {
        Container(transferAgent, "Transfer Agent", "Python", "Batches, checksums, resumable uploads to OSC")
        Container(tapisClient, "Tapis Client", "Python", "Creates jobs, tracks status, retries")
        Container(configStore, "Config & RBAC Store", "PostgreSQL", "Stores AOIs, policies, roles, audit logs")
    }

    Rel(cameraTrap, eventFilter, "Sends scored detections", "In-process/IPC")
    Rel(eventFilter, ringBuffer, "Writes events & media", "Filesystem API")
    Rel(eventFilter, flightPlanner, "Triggers plan generation", "MQTT/HTTP")
    Rel(flightPlanner, configStore, "Fetches AOI polygons", "SQL")
    Rel(flightPlanner, droneController, "Sends mission plan", "JSON/SDK")
    Rel(droneController, ringBuffer, "Stores captured media", "Filesystem API")
    Rel(ringBuffer, transferAgent, "Queues batches for upload", "Queue")
    Rel(transferAgent, tapisClient, "Notifies transfer complete", "HTTP/Internal")
    Rel(tapisClient, configStore, "Fetches job templates", "SQL")

    UpdateRelStyle(cameraTrap, eventFilter, $offsetY="-20")
    UpdateRelStyle(eventFilter, ringBuffer, $offsetX="-50")
    UpdateRelStyle(flightPlanner, droneController, $offsetY="-20")
```
````

**Caption:**  
Eight containers span edge (camera trap, event filter, ring buffer), drone (planner, controller), and cloud/transfer (agent, Tapis client, config store). Data flows from detection to scoring, event filtering, flight planning, drone capture, buffering, transfer, and job submission.

**Container Responsibilities:**

### Edge Layer
- **Camera Trap Agent:** Motion-triggered capture, on-device ML inference (≤2s/frame), perceptual hashing for deduplication
- **Event Filter Service:** Applies species/confidence thresholds, temporal coalescing (30s windows), rate limiting, alert generation
- **Ring Buffer Store:** Configurable retention (age/size), atomic writes, purge on delivery confirmation

### Drone Layer
- **Flight Planner:** Wind-adaptive lawnmower patterns, battery feasibility checks, AOI-bounded waypoint generation, mission segmentation
- **Drone Controller:** Parrot Anafi SDK integration, geofence enforcement, RTH failsafe, real-time telemetry logging

### Cloud/Transfer Layer
- **Transfer Agent:** Chunked multipart upload (5MB chunks), checksum verification, exponential backoff on failures, adaptive throttling
- **Tapis Client:** Job template instantiation, polling with backoff, retry logic (max 3 attempts), status persistence
- **Config & RBAC Store:** Versioned AOIs/policies, role-based access control, immutable audit logs (180 days)

**Technology Choices:**
- Python for portability and ML library support (TensorFlow Lite, OpenCV)
- PostgreSQL for ACID compliance and geospatial queries (PostGIS)
- MQTT for low-bandwidth edge messaging with offline buffering
- Parrot SDK for certified drone control with safety guarantees

