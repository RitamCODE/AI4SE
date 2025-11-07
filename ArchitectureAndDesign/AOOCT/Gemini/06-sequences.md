sequenceDiagram
    actor Labeler
    participant WebUI as Web UI
    participant APIGW as API Gateway
    participant DB as Metadata Database
    participant S3 as Object Storage

    Labeler->>WebUI: Opens a case for review
    WebUI->>APIGW: GET /cases/{id}/artifacts
    APIGW->>S3: Generate presigned URL for mask file
    S3-->>APIGW: URL
    APIGW-->>WebUI: Returns artifact list with URLs

    WebUI->>S3: Fetches mask file
    WebUI-->>Labeler: Displays image and mask overlay

    Labeler->>WebUI: Edits the mask using UI tools
    WebUI->>APIGW: POST /labels {case_id, edits} (Idempotency-Key: <uuid>)
    APIGW->>S3: PUT new_mask_version.nii.gz
    APIGW->>DB: INSERT INTO labels (user_id, case_id, old_mask_ref, new_mask_ref)
    APIGW->>DB: INSERT INTO audit_log (action='label_edit', user_id, case_id)
    DB-->>APIGW: Success
    APIGW-->>WebUI: 201 Created {LABEL_ID}
    WebUI-->>Labeler: Shows "Saved" confirmation
  sequenceDiagram
    actor MLE as ML Engineer
    participant WebUI as Web UI
    participant APIGW as API Gateway
    participant ModelRegistry as Model Registry Service
    participant Orchestrator as Pipeline Orchestrator

    MLE->>WebUI: Registers new model version with card
    WebUI->>APIGW: POST /models {model_card}
    APIGW->>ModelRegistry: registerModel(card)
    ModelRegistry-->>APIGW: 201 Created {MODEL_ID}

    MLE->>WebUI: Promote model to 'shadow'
    WebUI->>APIGW: PUT /models/{id}/state {state: 'shadow'}
    APIGW->>ModelRegistry: setModelState('shadow')

    Note over Orchestrator: On next relevant run trigger...
    Orchestrator->>ModelRegistry: getActiveModels()
    ModelRegistry-->>Orchestrator: {prod: model_v1, shadow: model_v2}
    Orchestrator->>Orchestrator: Dispatches two jobs: one for prod, one for shadow. Shadow result path is separate.

    loop After sufficient evaluation
        MLE->>WebUI: Promote model to 'production'
        WebUI->>APIGW: PUT /models/{id}/state {state: 'production'}
        APIGW->>ModelRegistry: setModelState('production')
    end