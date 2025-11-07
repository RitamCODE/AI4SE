# C4: System Context

```mermaid
graph TD
    subgraph System Boundary[AO-OCT Microstructure Analytics System]
        AOCTMAS[AO-OCT-MAS]
    end

    %% External Actors/Systems
    Clinician[Clinician\n(Reviews metrics, overlays)] -->|"Uploads cases, reviews results"\n"REST API (A-01, A-3)"| AOCTMAS
    Researcher[Researcher\n(Analyzes cohorts, exports data)] -->|"Triggers runs, exports metrics"\n"REST API (A-02, A-04, A-06)"| AOCTMAS
    ImagingTech[Imaging Technician\n(Monitors runs, re-queues jobs)] -->|"Monitors pipeline, re-queues failed runs"\n"REST API (A-03)"| AOCTMAS
    Labeler[Labeler/Adjudicator\n(Edits masks, resolves conflicts)] -->|"Submits mask edits, adjudicates"\n"REST API (A-05), Review UI (U-02)"| AOCTMAS
    MLEngineer[ML Engineer\n(Trains, deploys models)] -->|"Registers models, promotes versions"\n"Model Registry UI (U-03), API (A-06)"| AOCTMAS
    Admin[Admin\n(Manages access, audit)] -->|"Configures RBAC, audits logs"\n"SSO/RBAC (FR-070)"| AOCTMAS
    DataManager[Data Manager\n(De-identifies, schedules exports)] -->|"De-identifies data, schedules exports"\n"Export API (FR-080)"| AOCTMAS
    QA[QA\n(Runs regression tests)] -->|"Executes golden-case suite"\n"CI/CD (VV-02)"| AOCTMAS
    Auditor[Auditor\n(Checks compliance)] -->|"Exports audit logs"\n"Audit API (FR-071)"| AOCTMAS

    %% External Systems
    PACSScanner[PACS/Scanner\n(Source of AO-OCT volumes)] -->|"Sends DICOM/OCT-BIN"\n"DICOM C-STORE (D-01), File Drop (D-02)"| AOCTMAS
    ObjectStorage[S3-Compatible Storage\n(Stores raw/derived data)] <--|"Stores/retrieves volumes, masks, metrics"\n"S3 API (FR-004)"| AOCTMAS
    MetadataDB[Postgres\n(Indexes runs, provenance)] <--|"Stores run manifests, audit logs"\n"SQL (FR-003, FR-071)"| AOCTMAS
    ModelRegistry[Model Registry\n(Stores model versions)] <--|"Stores model cards, weights"\n"REST API (FR-060)"| AOCTMAS
    SSOProvider[SSO Provider\n(Authenticates users)] -->|"Provides auth tokens"\n"OAuth2/OIDC (FR-070)"| AOCTMAS
    ResearchShare[Research Data Share\n(Receives exports)] <--|"Receives de-identified exports"\n"S3/NFS (FR-080)"| AOCTMAS
