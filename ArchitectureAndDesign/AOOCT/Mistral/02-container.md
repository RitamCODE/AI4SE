
---

### 2. `02-container.md`
```markdown
# C4: Container

```mermaid
graph TD
    subgraph AOCTMAS[AO-OCT Microstructure Analytics System]
        %% Containers
        Ingest[Ingest Service\n(Python, FastAPI)\nResponsible for case registration, validation, and run triggering] -->|"POST /cases, /runs"\n"REST (A-01, A-02)"| Preproc
        Preproc[Preprocessing Service\n(Python, CUDA, PyTorch)\nResponsible for motion correction, speckle reduction, normalization] -->|"Volume + QC score"\n"Internal gRPC (FR-010, FR-011)"| Registration
        Registration[Registration Service\n(Python, CUDA, SimpleITK)\nResponsible for layer segmentation, ROI alignment] -->|"Aligned volume + transforms"\n"Internal gRPC (FR-020)"| Segmentation
        Segmentation[Segmentation Service\n(Python, CUDA, PyTorch/TensorRT)\nResponsible for cone/capillary/nerve fiber segmentation] -->|"Masks + uncertainty"\n"Internal gRPC (FR-030, FR-031, FR-032)"| Quantification
        Quantification[Quantification Service\n(Python, Pandas)\nResponsible for metric computation, QC flags] -->|"Metrics bundle"\n"Internal gRPC (FR-040)"| ReviewUI
        ReviewUI[Review UI\n(React, FastAPI)\nResponsible for mask editing, adjudication, visualization] -->|"Label edits"\n"REST (A-05), WebSocket (U-02)"| Quantification
        ModelRegistry[Model Registry\n(Python, FastAPI, MLflow)\nResponsible for model versioning, promotion, drift detection] -->|"Model weights + card"\n"REST (A-06), gRPC (FR-060)"| Segmentation
        Export[Export Service\n(Python, FastAPI)\nResponsible for de-identification, scheduled exports] -->|"De-identified metrics"\n"S3/NFS (FR-080)"| ResearchShare
        Audit[Audit Service\n(Python, FastAPI)\nResponsible for access logging, compliance checks] -->|"Audit logs"\n"REST (FR-071)"| MetadataDB

        %% Shared Infrastructure
        ObjectStorage[(Object Storage\nS3-Compatible)] <--|"Stores volumes, masks, metrics"\n"S3 API (FR-004)"| Ingest
        ObjectStorage <--|"Stores run artifacts"\n"S3 API (FR-004)"| Preproc
        ObjectStorage <--|"Stores aligned volumes"\n"S3 API (FR-004)"| Registration
        ObjectStorage <--|"Stores masks, overlays"\n"S3 API (FR-004)"| Segmentation
        ObjectStorage <--|"Stores metrics, manifests"\n"S3 API (FR-004)"| Quantification
        MetadataDB[(Metadata DB\nPostgres)] <--|"Indexes run manifests"\n"SQL (FR-003)"| Ingest
        MetadataDB <--|"Logs provenance, audit"\n"SQL (FR-071)"| Audit
        ModelRegistryDB[(Model Registry DB\nPostgres)] <--|"Stores model cards"\n"SQL (FR-060)"| ModelRegistry
    end
