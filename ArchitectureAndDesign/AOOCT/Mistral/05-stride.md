# STRIDE Summary

| Boundary (C4 Node/Edge)       | Threat                          | Impact                          | Mitigation                                      | Test Hook                     | Linked NFR/ADR          |
|--------------------------------|----------------------------------|----------------------------------|-------------------------------------------------|-------------------------------|--------------------------|
| Ingest ← PACS/Scanner          | Tampered input volumes           | Incorrect metrics, misdiagnosis  | Checksum validation (FR-002), DICOM signing    | Golden-case regression (VV-02)| NFR-S-01, ADR-002       |
| ReviewUI → Labeler             | CSRF/XSS in edit submissions     | Mask corruption, data loss      | CSP headers, CSRF tokens (FR-070)               | Penetration test              | NFR-S-01, ADR-003       |
| ModelRegistry ← MLEngineer     | Malicious model upload          | Biased/invalid results           | Model card validation, sandboxed inference    | Metric drift alert (FR-062)   | NFR-A-01, ADR-004       |
| Export → ResearchShare         | PHI leakage in exports          | Compliance violation            | Automated DLP scan (FR-072), field redaction   | Audit log review (FR-071)     | NFR-S-02, ADR-005       |
| Audit → MetadataDB             | Log injection                    | False audit trails               | Parameterized queries, RBAC (FR-070)            | Log integrity check           | NFR-O-01                |
| Preproc ↔ ObjectStorage       | Artifact spoofing               | Pipeline poisoning              | S3 object locking, immutable manifests (FR-004) | Manifest checksum validation  | ADR-002                 |
| SSOProvider → All Containers   | Token hijacking                  | Unauthorized access              | Short-lived JWT, RBAC enforcement (FR-070)      | Failed login alert            | NFR-S-01                |
