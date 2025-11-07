***

### 02-container.md

```markdown
### C4: Container

A Container diagram showing the high-level components within the AO-OCT-MAS.

```mermaid
graph TD
    actor "User"

    subgraph "AO-OCT Microstructure Analytics System"
        boundary ---
        Container_WebApp("
        **Web UI**
        [React SPA]
        Provides data visualization, label editing, model management, and administrative dashboards.
        ")

        Container_APIGW("
        **API Gateway**
        [NGINX / Kong]
        Handles authentication, request routing, and serves as the single entry point for all API calls.
        ")

        Container_Orchestrator("
        **Pipeline Orchestrator**
        [Argo Workflows / K8s]
        Manages the DAG of processing tasks for each run, handling scheduling, retries, and artifact passing.
        ")

        Container_Workers("
        **Processing Workers**
        [Python/PyTorch Container]
        GPU-enabled containers that execute individual pipeline steps (e.g., preprocessing, segmentation).
        ")

        Container_ModelRegistry("
        **Model Registry Service**
        [MLflow / Custom Service]
        Manages model lifecycle, versions, metadata (cards), and deployment gates (canary/shadow).
        ")

        Container_Ingest("
        **Data Ingest Service**
        [Python / Go Service]
        Listens for new data via file drops or DICOM, validates it, and initiates processing runs.
        ")

        Container_DB("
        **Metadata Database**
        [PostgreSQL]
        Stores case info, run manifests, metrics, model metadata, labels, and audit logs.
        ")

        Container_Storage("
        **Object Storage**
        [S3 / MinIO]
        Stores raw data volumes, intermediate artifacts, segmentation masks, and model weights.
        ")
    end

    User -- "Interacts via HTTPS" --> Container_WebApp
    Container_WebApp -- "Makes API calls via HTTPS/REST" --> Container_APIGW

    subgraph External Systems
        System(Scanner / PACS)
        System(SSO Provider)
    end

    System -- "Sends data via DICOM/NFS" --> Container_Ingest
    Container_APIGW -- "Validates token with SSO Provider" --> SSO_Provider

    Container_Ingest -- "Writes files via S3 API" --> Container_Storage
    Container_Ingest -- "Registers case via HTTPS/REST" --> Container_APIGW

    Container_APIGW -- "Triggers runs, submits labels via gRPC/REST" --> Container_Orchestrator
    Container_APIGW -- "Manages models via gRPC/REST" --> Container_ModelRegistry
    Container_APIGW -- "Reads/writes metadata via SQL" --> Container_DB

    Container_Orchestrator -- "Schedules jobs" --> Container_Workers
    Container_Workers -- "Read/write artifacts via S3 API" --> Container_Storage
    Container_Workers -- "Updates status/metrics via SQL" --> Container_DB

    Container_ModelRegistry -- "Reads/writes model info via SQL" --> Container_DB
    Container_ModelRegistry -- "Reads model weights via S3 API" --> Container_Storage