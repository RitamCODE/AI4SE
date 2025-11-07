
### Architecture Decision Records (ADRs)

1.  **ADR-1: Edge Event Filtering Strategy**
    * **Problem/Context:** Edge devices are resource-constrained (compute, storage) and operate on intermittent networks. Sending every detected frame is not feasible and would overwhelm the network (spam). (SRS FR-A, FR-B, NFR-P1, US-03, US-36).
    * **Options:**
        1.  **Simple Thresholding:** Only send if `confidence > X`.
        2.  **Temporal Coalescing:** (SRS FR-007) Group detections within a time window (e.g., 60s) into a single "Event" with summary stats.
        3.  **Perceptual Hashing (Deduplication):** (SRS FR-005) Compute a pHash for detected frames and discard if the hash is too similar to recently sent frames.
    * **Trade-offs:**
        * *Option 1:* Simple, low-compute. Fails to prevent spam from a lingering animal.
        * *Option 2:* Good balance. Low-compute. Effectively reduces "spam" (US-36). Might miss subtle new behavior within the window.
        * *Option 3:* Most accurate for "near-identical" frames. Has a higher compute cost, which may violate NFR-P1 (≤ 2s/frame).
    * **Decision:** Implement **Temporal Coalescing (Option 2)** as the primary "Must" (M) requirement. This directly addresses FR-007 and US-36. **Perceptual Hashing (Option 3)** will be used as a secondary filter on the *exemplar* frames chosen by coalescing, as specified in FR-005.
    * **Revisit:** If NFR-P1 is consistently violated by the hashing step, or if coalescing alone still produces too many redundant events.
    * **Links:** C4: `Edge Agent`. Stories: US-03, US-36.

2.  **ADR-2: Data Transfer Mechanism to OSC**
    * **Problem/Context:** Media batches must be transferred reliably from edge devices (intermittent, low-bandwidth) to OSC storage. Transfers must be resumable and verifiable. (SRS FR-E, NFR-P3, NFR-R1, US-03, US-21, US-37).
    * **Options:**
        1.  **Standard HTTPS POST:** Simple `curl` or `requests` upload per file.
        2.  **Rsync over SSH:** Battle-tested, delta-transfers. Requires SSH setup and key management.
        3.  **Chunked, Resumable HTTPS (TUS.io / S3-Multipart):** Standard protocol for resumable uploads. Requires client/server support.
        4.  **Tapis Files API:** Use the Tapis platform's built-in file transfer mechanisms, as Tapis is already a dependency (SRS 3.3).
    * **Trade-offs:**
        * *Option 1:* Fails on disconnect. No resume. Violates FR-019 and NFR-R1.
        * *Option 2:* Resumable and robust, but SSH may be "chatty" on high-latency links and adds key management overhead.
        * *Option 3:* Solves the core problem. The `Data Transfer Service` must manage checkpointing state.
        * *Option 4:* Aligns with the project's dependency on Tapis. Assumes Tapis Files API provides resumable, checksummed transfers.
    * **Decision:** Use the **Tapis Files API (Option 4)** as the primary transfer mechanism. The `Data Transfer Service` will act as a client to this API, handling local batching (FR-019), adaptive throttling (FR-020), and verifying checksums end-to-end (FR-019). This consolidates dependencies.
    * **Revisit:** If the Tapis Files API does not support resumable transfers or performs poorly, investigate a hybrid approach (e.g., Option 3 to S3, then trigger Tapis).
    * **Links:** C4: `Data Transfer Service` -> `OSC Storage` / `Tapis API` edges. Stories: US-21, US-37.

3.  **ADR-3: Mission Approval Workflow**
    * **Problem/Context:** Drone flights have significant safety and regulatory implications (US-33). The system must balance the goal of automation (NFR-P2) with the need for human-in-the-loop safety (US-08).
    * **Options:**
        1.  **Always Manual Approval:** Every planned mission is queued. A `Drone Operator` must log in and click "Approve".
        2.  **Fully Automatic:** As soon as an event triggers a plan, launch immediately.
        3.  **Policy-Based Auto-Launch:** Define rules (e.g., daylight, wind < X, pre-approved AOI). If rules pass, auto-launch. If not, queue for manual approval.
    * **Trade-offs:**
        * *Option 1:* Safest. Meets US-08. Fails on "reducing manual intervention" goal and NFR-P2 (≤ 5 min latency).
        * *Option 2:* Fastest. Violates safety/compliance (FR-014, US-33).
        * *Option 3:* Explicitly required by SRS FR-014 ("operator approval OR policy-based auto-launch"). Balances speed (NFR-P2) with safety (US-08, US-09).
    * **Decision:** Implement **Policy-Based Auto-Launch (Option 3)**. The `Web Portal & API` will manage these policies. The `Mission Planner` will evaluate them. If a policy is not met (or none exists), the mission is queued for manual approval by a `Drone Operator` via the `Web Portal & API`.
    * **Revisit:** If auto-launch policies prove too complex or if safety incidents occur, revert to manual-only approval (Option 1) for high-risk AOIs.
    * **Links:** C4: `Mission Planner`, `Web Portal & API`. Stories: US-08, US-09, US-12, US-33. NFR: NFR-P2.