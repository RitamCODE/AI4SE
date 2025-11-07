# Design: Sequence Diagrams

## Diagram 1: Upload Volume and Trigger Run

```mermaid
sequenceDiagram
    participant U as User (Researcher)
    participant UI as Web UI
    participant API as API Gateway
    participant DB as Metadata DB
    participant S3 as Object Store
    participant Orch as Orchestrator

    U->>UI: Upload OCT volume + metadata
    UI->>API: POST /cases {file, device_id, consent}
    API->>API: Validate PHI policy, de-ID if needed
    API->>S3: Store volume (versioned, KMS-encrypted)
    S3-->>API: S3 URI + checksum
    API->>DB: INSERT case (id, s3_uri, checksum, user_id)
    DB-->>API: CASE_ID
    API-->>UI: 201 {case_id}
    UI-->>U: Display CASE_ID

    U->>UI: Click "Run Pipeline"
    UI->>API: POST /runs {case_id, pipeline_config}
    API->>DB: Check case exists & user authorized
    DB-->>API: Case metadata
    API->>Orch: Trigger run (case_id, config, user_id)
    Orch->>DB: INSERT run (id=RUN_ID, state=QUEUED, manifest={input_checksum, code_ref, seed})
    DB-->>Orch: RUN_ID
    Orch-->>API: 202 {run_id}
    API-->>UI: 202 {run_id}
    UI-->>U: Display "Run queued: RUN_ID"

    Note over Orch: Schedule preproc → segment → metrics
    Orch->>DB: UPDATE run state=RUNNING
    Orch->>Orch: Retry on transient failure (max 3, exp backoff)
    alt Success
        Orch->>DB: UPDATE run state=SUCCEEDED, artifacts_uri
    else Failure
        Orch->>DB: UPDATE run state=FAILED, error_log
    end

    U->>UI: Poll GET /runs/{run_id}
    UI->>API: GET /runs/{run_id}
    API->>DB: SELECT run by id
    DB-->>API: {state, artifacts_uri, logs}
    API-->>UI: 200 {state, artifacts}
    UI-->>U: Show status + download links
```

**Annotations**:
- **Idempotency**: Re-POST same `case_id + config` returns existing `RUN_ID` if manifest identical.
- **Timeout**: Orchestrator jobs have 1h deadline; auto-fail and log.
- **Error**: API returns 4xx for validation, 503 if Orchestrator unavailable.

**Stories**: Upload volume, RUN_ID tracking, idempotent retries  
**SRS**: FR-001, FR-002, FR-003, FR-004, NFR-R-01

---

## Diagram 2: Review and Edit Mask with Active Learning

```mermaid
sequenceDiagram
    participant U as User (Labeler)
    participant UI as Web UI
    participant API as API Gateway
    participant S3 as Object Store
    participant DB as Metadata DB

    U->>UI: Open review tool
    UI->>API: GET /active-learning/queue?uncertainty_threshold=0.3
    API->>DB: SELECT tiles WHERE uncertainty > 0.3 ORDER BY uncertainty DESC LIMIT 50
    DB-->>API: Tile list [{tile_id, run_id, uncertainty, s3_uri}]
    API-->>UI: 200 {tiles}
    UI-->>U: Display ranked tile list

    U->>UI: Click tile_123
    UI->>API: GET /tiles/tile_123 (raw + mask + uncertainty)
    API->>S3: Fetch tile PNG + mask PNG + uncertainty map
    S3-->>API: Binary data (cached)
    API-->>UI: 200 {raw_img, mask_img, uncertainty_map}
    UI-->>U: Render overlay with <100ms (TanStack Query cache)

    U->>UI: Edit mask (brush tool)
    UI->>UI: Optimistic update (local state)
    U->>UI: Click "Save"
    UI->>API: POST /labels {tile_id, edited_mask_blob, comment}
    API->>API: Compute diff (original vs edited)
    API->>S3: Store new mask version (v2, timestamped)
    S3-->>API: New S3 URI
    API->>DB: INSERT label (id=LABEL_ID, tile_id, user_id, s3_uri, diff, created_at)
    API->>DB: INSERT audit_log (action=LABEL_EDIT, user_id, tile_id, timestamp)
    DB-->>API: LABEL_ID
    API-->>UI: 201 {label_id, diff_preview}
    UI-->>U: Show "Saved v2, diff: +12px -5px"

    alt Rollback requested
        U->>UI: Click "Rollback to v1"
        UI->>API: POST /labels/rollback {label_id, target_version=1}
        API->>DB: INSERT new label record pointing to v1 S3 URI
        API->>DB: INSERT audit_log (action=ROLLBACK)
        DB-->>API: New LABEL_ID
        API-->>UI: 200 {restored_label_id}
        UI-->>U: "Restored to v1"
    end
```

**Annotations**:
- **Tile fetch**: Cached client-side (TanStack Query); 512×512 PNG ~100KB → <100ms (NFR-U-01).
- **Optimistic UI**: Mask overlay updates instantly; server confirmation async.
- **Versioning**: Each edit creates new label record; diffs stored as JSON.

**Stories**: Edit mask, active-learning queue, overlay viz  
**SRS**: FR-050, FR-051, U-02, NFR-U-01

---

## Diagram 3: Register Model, Canary Deploy, Drift Alert

```mermaid
sequenceDiagram
    participant U as User (ML Engineer)
    participant UI as Web UI
    participant API as API Gateway
    participant MR as Model Registry
    participant S3 as Object Store
    participant DB as Metadata DB
    participant Seg as Segmenter
    participant Cron as Drift Cron Job

    U->>UI: Upload model weights + card
    UI->>API: POST /models {weights_blob, card={data_hash, metrics, risks}}
    API->>S3: Store weights (KMS-encrypted)
    S3-->>API: S3 URI + checksum
    API->>MR: Register model (s3_uri, card, checksum)
    MR->>DB: INSERT model (id=MODEL_ID, version, card_json, checksum, stage=STAGING)
    DB-->>MR: MODEL_ID
    MR-->>API: 201 {model_id, stage=STAGING}
    API-->>UI: 201 {model_id}
    UI-->>U: "Model registered, ID=MODEL_123, stage=STAGING"

    U->>UI: Click "Promote to Canary (1% traffic)"
    UI->>API: POST /models/{model_id}/promote {stage=CANARY, traffic_pct=1}
    API->>MR: Check promotion gates (F1 ≥ 0.90, ECE ≤ 0.05)
    MR->>DB: SELECT model metrics
    DB-->>MR: {f1: 0.92, ece: 0.03}
    alt Gates pass
        MR->>DB: UPDATE model stage=CANARY, traffic=1%
        MR->>API: Feature flag: route 1% to MODEL_ID
        API-->>UI: 200 "Canary active"
        UI-->>U: "Canary live, monitoring..."
    else Gates fail
        MR-->>API: 400 "F1 below threshold"
        API-->>UI: 400 {error}
        UI-->>U: "Promotion blocked"
    end

    Note over Seg: Production inference uses canary routing
    Seg->>API: GET /models/active?traffic_slot=17 (hash to 1%)
    API->>MR: Resolve model for slot
    MR-->>API: MODEL_ID (canary) or PROD_MODEL_ID
    API-->>Seg: {model_id, s3_weights_uri}
    Seg->>S3: Download weights (if not cached)
    S3-->>Seg: Weights blob
    Seg->>Seg: Load model, run inference

    Note over Cron: Weekly drift check
    Cron->>DB: SELECT predictions, ground_truth FROM last_week
    DB-->>Cron: Population stats
    Cron->>Cron: Compute ECE, compare to baseline
    alt Drift detected (ECE > 0.05 + margin)
        Cron->>API: POST /alerts {type=DRIFT, model_id, ece_delta}
        API->>DB: INSERT alert, notify ML Engineer
        DB-->>API: Alert logged
        API-->>U: Email/Slack "Drift alert: MODEL_123 ECE=0.08"
    else No drift
        Cron->>DB: Log weekly stats (no alert)
    end
```

**Annotations**:
- **Promotion gates**: Automated check against NFR-A-01, NFR-A-02; manual override requires admin role.
- **Shadow traffic**: API Gateway routes 1% via consistent hash (user_id or run_id mod 100).
- **Rollback**: POST `/models/{id}/rollback` sets traffic to 0%, reverts to last PROD model (one-click).

**Stories**: Register model, canary deploys, drift alerts  
**SRS**: FR-060, FR-061, FR-062, NFR-A-01, NFR-A-02