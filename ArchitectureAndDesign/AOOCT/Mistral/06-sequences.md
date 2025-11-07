
---

### 6. `06-sequences.md`
```markdown
# Design: Sequence Diagrams

## 1. Pipeline Run (Researcher Uploads Volume)
```mermaid
sequenceDiagram
    participant Researcher
    participant Ingest
    participant Preproc
    participant Registration
    participant ObjectStorage
    participant MetadataDB

    Researcher->>Ingest: POST /cases (volume + metadata)
    Ingest->>ObjectStorage: Store raw volume (S3 PUT)
    Ingest->>MetadataDB: Log case (SQL INSERT)
    Researcher->>Ingest: POST /runs (params, model_id)
    Ingest->>MetadataDB: Create RUN manifest (FR-003)
    Ingest->>Preproc: Trigger preprocessing (gRPC)
    Preproc->>ObjectStorage: Fetch volume (S3 GET)
    Preproc->>Preproc: Motion correction (FR-010)
    Preproc->>ObjectStorage: Store QC snapshot (S3 PUT)
    Preproc->>Registration: Send aligned volume (gRPC)
    Registration->>ObjectStorage: Store transforms (S3 PUT)
    Registration->>MetadataDB: Update RUN status (SQL UPDATE)
    MetadataDB-->>Researcher: RUN_ID (202 Accepted)
sequenceDiagram
    participant Labeler
    participant ReviewUI
    participant Quantification
    participant ObjectStorage
    participant MetadataDB

    Labeler->>ReviewUI: Open tile editor (WebSocket)
    ReviewUI->>ObjectStorage: Fetch mask/uncertainty (S3 GET)
    Labeler->>ReviewUI: Submit edits (POST /labels)
    ReviewUI->>ObjectStorage: Store new mask version (S3 PUT)
    ReviewUI->>Quantification: Recompute metrics (gRPC)
    Quantification->>MetadataDB: Log LABEL_ID + diff (FR-050)
    MetadataDB-->>Labeler: Confirmation (201 Created)
    opt Conflict
        Labeler->>ReviewUI: Request adjudication
        ReviewUI->>MetadataDB: Flag for multi-rater review (FR-052)
    end
sequenceDiagram
    participant MLEngineer
    participant ModelRegistry
    participant Segmentation
    participant MetadataDB

    MLEngineer->>ModelRegistry: POST /models (weights + card)
    ModelRegistry->>MetadataDB: Store model card (FR-060)
    ModelRegistry->>Segmentation: Shadow deploy (10% traffic)
    Segmentation->>ModelRegistry: Stream metrics (gRPC)
    alt Metrics pass gates
        MLEngineer->>ModelRegistry: Promote to production
        ModelRegistry->>Segmentation: Update active model
    else Metrics fail
        MLEngineer->>ModelRegistry: Rollback
        ModelRegistry->>Segmentation: Revert to prior version
    end
