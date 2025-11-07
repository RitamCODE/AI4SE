# C4: Container

```mermaid
C4Container
    title Container View - AO-OCT Microstructure Analytics System

    Person(user, "User", "Researcher, Tech, Labeler, ML Engineer, Admin")
    System_Ext(sso, "SSO Provider")
    System_Ext(scanner, "Scanner/PACS")
    System_Ext(storage, "Export Storage")

    Container(webui, "Web UI", "React", "Upload, review, visualize overlays/metrics")
    Container(apigateway, "API Gateway", "FastAPI/Kong", "REST endpoints, auth, rate limiting")
    Container(orchestrator, "Orchestrator", "Temporal/Airflow", "Manages pipeline runs, retries, state")
    Container(preprocessor, "Preprocessor", "Python/CUDA", "Motion correction, speckle reduction, normalization")
    Container(segmenter, "Segmenter", "PyTorch/TensorRT", "Cones, capillaries, nerve fibers + uncertainty")
    Container(metrics, "Metrics Engine", "Python/Pandas", "Computes densities, diameters, CI, QC flags")
    Container(modelregistry, "Model Registry", "MLflow/Custom", "Model cards, promotion, drift detection")
    Container(db, "Metadata DB", "PostgreSQL", "Cases, runs, labels, audit logs")
    Container(objectstore, "Object Store", "S3/MinIO", "Volumes, masks, manifests, artifacts")

    Rel(user, webui, "Uses", "HTTPS")
    Rel(webui, apigateway, "Calls", "REST/JSON")
    Rel(apigateway, sso, "Validates token", "OAuth2")
    Rel(apigateway, orchestrator, "Triggers runs", "gRPC/REST")
    Rel(apigateway, db, "Queries metadata", "SQL")
    Rel(apigateway, objectstore, "Fetches artifacts", "S3 API")
    Rel(scanner, apigateway, "Pushes volumes", "REST/DICOM")
    
    Rel(orchestrator, preprocessor, "Schedules job", "Task queue")
    Rel(orchestrator, segmenter, "Schedules job", "Task queue")
    Rel(orchestrator, metrics, "Schedules job", "Task queue")
    Rel(orchestrator, db, "Updates run state", "SQL")
    
    Rel(preprocessor, objectstore, "Reads/writes volumes", "S3 API")
    Rel(segmenter, objectstore, "Reads volumes, writes masks", "S3 API")
    Rel(segmenter, modelregistry, "Loads model weights", "REST")
    Rel(metrics, objectstore, "Reads masks, writes metrics", "S3 API")
    Rel(metrics, db, "Stores metrics", "SQL")
    
    Rel(modelregistry, objectstore, "Stores weights", "S3 API")
    Rel(modelregistry, db, "Stores model metadata", "SQL")
    
    Rel(apigateway, storage, "Exports metrics", "S3/NFS")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="2")
```

**Caption**: Eight containers orchestrate the pipeline. API Gateway enforces auth/RBAC. Orchestrator schedules idempotent jobs. Preprocessor, Segmenter, Metrics Engine are GPU-accelerated workers. Model Registry manages ML lifecycle. PostgreSQL and S3 provide persistence.