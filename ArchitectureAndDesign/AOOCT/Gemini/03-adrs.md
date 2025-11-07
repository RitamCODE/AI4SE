***

### 03-adrs.md

```markdown
### Architecture Decision Records (ADRs)

* **ADR-01: Pipeline Orchestration Engine**
    * **Problem/Context**: The system requires a robust mechanism to execute multi-step, computationally intensive ML pipelines on containerized infrastructure. This includes managing dependencies, handling transient failures with retries, and ensuring provenance is captured for each run. (SRS 2.1, 4.2-4.5; Stories: "As a researcher I want to upload...", "As an operator I want retries...")
    * **Options**:
        1.  **Kubernetes-native Workflow Engine (e.g., Argo Workflows)**: Define pipelines as Kubernetes CRDs, leveraging native K8s features for scheduling, volumes, and secrets.
        2.  **Library-based Orchestrator (e.g., Prefect, Dagster)**: Define pipelines in Python code, with a separate agent/scheduler deploying jobs to Kubernetes.
        3.  **Custom Queue-based System**: Use a message queue (e.g., RabbitMQ) and custom workers (e.g., Celery) to manage tasks.
    * **Trade-offs**:
        * *Argo Workflows* has a steeper learning curve but offers tight integration with the target K8s environment (SRS 2.3), excellent observability, and native handling of artifacts and complex DAGs.
        * *Prefect/Dagster* offer a more developer-friendly, code-first experience but add another layer of abstraction and a separate control plane to manage.
        * A *custom system* provides maximum control but incurs significant development and maintenance overhead, "reinventing the wheel" for features like retries and DAG management (violates NFR-M-02 maintainability principle).
    * **Decision**: **Option 1: Argo Workflows**. The choice aligns with the specified Kubernetes operating environment (SRS 2.3), provides mature features for idempotent runs (NFR-R-01) and manifest creation (FR-003), and is well-suited for ML operations.
    * **Revisit**: If the system must be deployed outside of Kubernetes (e.g., on Docker Compose only), a library-based orchestrator might become more suitable.

* **ADR-02: Data and Artifact Storage**
    * **Problem/Context**: The system must store raw PHI, derived non-PHI artifacts, and immutable run manifests securely and traceably, with support for versioning. (SRS 2.1, 4.1, C-01, C-02, DR-03)
    * **Options**:
        1.  **Single Versioned Object Store Bucket**: Use one S3-compatible bucket with versioning enabled and strict, path-based IAM policies to segregate data types (e.g., `/raw-phi/`, `/derived/`, `/manifests/`).
        2.  **Multiple Segregated Buckets**: Use separate buckets for raw PHI, derived data, and logs, each with its own distinct security policy.
        3.  **Network File System (NFS) + Database Blobs**: Store files on an NFS share and manage metadata and smaller artifacts as BLOBs in the database.
    * **Trade-offs**:
        * *Single Bucket* simplifies infrastructure management and cross-data lookups. Path-based policies can effectively enforce security boundaries. Versioning directly supports audit and rollback requirements (FR-050, FR-004).
        * *Multiple Buckets* provides stronger security isolation (blast radius reduction) but increases operational complexity and can complicate cross-domain operations (e.g., linking a manifest to its raw data).
        * *NFS* is often harder to secure, scale, and manage for high-throughput I/O compared to object storage. Storing BLOBs in the DB can lead to performance degradation (NFR-P-01).
    * **Decision**: **Option 1: Single Versioned Object Store Bucket**. This approach provides the best balance of security (via KMS encryption per NFR-S-01 and IAM policies), traceability (versioning for FR-004), and operational simplicity. Metadata will be indexed in the PostgreSQL database for fast queries.
    * **Revisit**: If a regulatory audit mandates physical or account-level separation of PHI data.

* **ADR-03: Model Deployment and Validation Strategy**
    * **Problem/Context**: New segmentation models must be introduced into the production environment safely, allowing for comparison against existing models without disrupting service and ensuring performance meets standards before full rollout. (SRS 4.7, FR-061, NFR-A-01, NFR-A-02)
    * **Options**:
        1.  **Blue/Green Deployment**: Maintain two identical production environments. Deploy the new model to the inactive environment, test, and then switch traffic.
        2.  **Canary Release**: Route a small subset of live traffic (e.g., 5% of runs) to the new model version and monitor performance. Gradually increase traffic.
        3.  **Shadow Deployment with Canary Promotion**: First, deploy the new model to receive copies of production traffic in a "shadow" mode (results are not returned to users but are analyzed). If metrics are acceptable, promote to a canary release.
    * **Trade-offs**:
        * *Blue/Green* is simple and provides fast rollback but is resource-intensive (doubles infrastructure cost) and doesn't allow for real-world performance comparison on the same traffic.
        * *Canary* is efficient but carries a risk of impacting a small subset of users if the new model is faulty.
        * *Shadow + Canary* offers the highest level of safety. Shadowing validates the model on live data with zero user impact (FR-061). The subsequent canary release confirms performance under load before full rollout. This is more complex to implement.
    * **Decision**: **Option 3: Shadow Deployment with Canary Promotion**. This phased approach provides the most robust validation against key accuracy (NFR-A-01) and reliability (NFR-R-01) requirements before full production use, directly supporting the "promotion gates" feature (FR-061).
    * **Revisit**: If the implementation complexity of traffic shadowing proves too high for the initial release schedule.