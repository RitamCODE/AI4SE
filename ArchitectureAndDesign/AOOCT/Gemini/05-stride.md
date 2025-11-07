***

### 05-stride.md

```markdown
### STRIDE Summary

A STRIDE threat model analysis for the system's external boundaries identified in the C4 Context diagram.

| Boundary | Threat | Impact | Mitigation | Test Hook | Linked NFR/ADR |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **User ↔ AO-OCT-MAS** | **S**poofing | Unauthorized user accesses/modifies PHI or system configuration. | Enforce strong authentication via SSO integration. | Penetration test with invalid/stolen tokens. | FR-070 |
| **User ↔ AO-OCT-MAS** | **T**ampering | Malicious user alters data in transit (e.g., edited mask, metrics). | Enforce TLS 1.2+ for all communication. Use checksums for uploads. | Man-in-the-middle attack simulation. | NFR-S-01 |
| **User ↔ AO-OCT-MAS** | **R**epudiation | User denies performing an action (e.g., deleting a case, approving a model). | Maintain immutable, detailed audit logs for all state-changing actions. | Verify audit log entries are generated for all critical API endpoints. | FR-071, FR-003 |
| **User ↔ AO-OCT-MAS** | **I**nformation Disclosure | An authenticated but unauthorized user accesses data from another project. | Enforce strict Role-Based Access Control (RBAC) at the API Gateway. | Test API endpoints with tokens from users with insufficient permissions. | FR-070 |
| **User ↔ AO-OCT-MAS** | **D**enial of Service | A user overwhelms the system with large uploads or excessive API calls. | Implement rate limiting and request size limits at the API Gateway. | Load test API endpoints to validate rate-limiting behavior. | NFR-R-02 (Assumed) |
| **User ↔ AO-OCT-MAS** | **E**levation of Privilege | A user exploits a vulnerability to gain higher permissions (e.g., from researcher to admin). | Apply principle of least privilege in RBAC roles. Regular dependency scanning. | Vulnerability scans and code reviews. | FR-070 |
| **Scanner/PACS → MAS** | **I**nformation Disclosure | PHI is intercepted during transmission from scanner to ingest service. | Use secure transport (TLS/IPsec VPN). Encrypt data at rest immediately upon ingest. | Network traffic analysis to ensure encryption. | NFR-S-01 |
| **MAS ↔ SSO Provider** | **T**ampering | Attacker intercepts and modifies SAML/OAuth assertion to impersonate a user. | Use signed and encrypted assertions; strictly validate issuer and audience. | Attempt to replay or modify assertions during authentication tests. | FR-070 |
| **MAS → Research Share** | **I**nformation Disclosure | De-identification process fails, leaking PHI into the research export. | Implement automated PHI redaction filters and a manual QA gate for exports. | Create golden test cases with known PHI to verify redaction logic. | FR-072, NFR-S-02 |
06-sequences.md
Markdown

### Design: Sequence Diagrams

#### 1. New Volume Ingestion and Processing

This diagram shows the success path for a user uploading a new volume and triggering a processing pipeline run.

```mermaid
sequenceDiagram
    actor User
    participant WebUI as Web UI
    participant APIGW as API Gateway
    participant Orchestrator as Pipeline Orchestrator
    participant Worker as Processing Worker
    participant DB as Metadata Database
    participant S3 as Object Storage

    User->>WebUI: Uploads volume file
    WebUI->>APIGW: POST /cases with metadata
    APIGW->>DB: INSERT INTO cases (metadata)
    DB-->>APIGW: Returns CASE_ID
    APIGW-->>WebUI: 201 Created {CASE_ID}
    WebUI-->>User: Displays Case ID, asks to start run

    User->>WebUI: Clicks "Start Run"
    WebUI->>APIGW: POST /runs {case_id, params}
    APIGW->>DB: Query for case details
    APIGW->>Orchestrator: submitWorkflow(run_spec)
    Orchestrator-->>APIGW: 202 Accepted {RUN_ID}
    APIGW-->>WebUI: Returns RUN_ID
    WebUI-->>User: Displays Run ID and "QUEUED" status

    Orchestrator->>Worker: Schedule job(run_spec)
    Worker->>DB: UPDATE runs SET status='RUNNING' WHERE id=RUN_ID
    Worker->>S3: GET raw_volume
    Worker->>Worker: Preproc -> Register -> Segment -> Quantify
    Worker->>S3: PUT derived_artifacts (masks, metrics.parquet)
    Worker->>DB: UPDATE runs SET status='SUCCEEDED', artifacts_path='...'