# STRIDE Summary

| **Boundary** | **Threat** | **Impact** | **Mitigation** | **Test Hook** | **Linked NFR/ADR** |
|--------------|------------|------------|----------------|---------------|--------------------|
| User → API Gateway | Spoofing (stolen credentials) | Unauthorized access | SSO + MFA; short-lived tokens; rate limiting | Pen-test with invalid tokens | NFR-S-01, ADR-004 |
| API Gateway → SSO | Tampering (MITM on auth flow) | Session hijack | TLS 1.2+; cert pinning | SSL Labs scan | NFR-S-01 |
| Scanner → API Gateway | Repudiation (volume upload denial) | Lost provenance | Audit log with scanner IP, timestamp, checksum | Verify audit trail completeness | FR-071, ADR-004 |
| API Gateway → Object Store | Information Disclosure (PHI leak) | HIPAA violation | KMS encryption; RBAC on buckets; no PHI in logs | Scan logs for PHI regex | NFR-S-01, ADR-003 |
| Segmenter → Model Registry | Tampering (malicious model weights) | Backdoor inference | Model checksum in registry; signature verification | Hash mismatch test | FR-060, ADR-002 |
| Web UI → API Gateway | Denial of Service (tile flood) | UI unresponsive | Rate limit per user; CDN caching; backpressure | Load test 1000 tile requests/sec | NFR-U-01, ADR-005 |
| Orchestrator → DB | Elevation of Privilege (SQL injection) | Metadata corruption | Parameterized queries; least-privilege DB roles | SQLMap scan | FR-070, ADR-004 |
| Metrics Engine → Object Store | Information Disclosure (manifest leak) | Algorithm disclosure | S3 bucket policies; pre-signed URLs with expiry | Attempt direct S3 access | NFR-S-01, ADR-003 |
| Export → Institutional Storage | Repudiation (deny data export) | Compliance gap | Manifest with checksum; signed export receipt | Verify signature on export file | FR-080, ADR-003 |

## Coverage Verification

### External Boundaries from C4 Context
- ✅ User → AO-OCT-MAS (covered: spoofing, DoS)
- ✅ AO-OCT-MAS → SSO (covered: tampering)
- ✅ Scanner → AO-OCT-MAS (covered: repudiation)
- ✅ AO-OCT-MAS → Institutional Storage (covered: repudiation)

### Critical Internal Boundaries from C4 Container
- ✅ API Gateway → Object Store (covered: information disclosure)
- ✅ Segmenter → Model Registry (covered: tampering)
- ✅ Orchestrator → DB (covered: elevation of privilege)
- ✅ Metrics Engine → Object Store (covered: information disclosure)

## Security Testing Requirements

1. **Penetration Testing**: Quarterly external pen-tests covering authentication, authorization, and API endpoints.
2. **Vulnerability Scanning**: Weekly automated scans with SQLMap, OWASP ZAP, and custom PHI regex checks.
3. **Load Testing**: Monthly stress tests simulating 1000+ concurrent tile requests and 100+ simultaneous run submissions.
4. **Compliance Audits**: Bi-annual HIPAA compliance reviews with independent auditors.
5. **Model Integrity**: Pre-deployment checksum verification for all model weights in CI/CD pipeline.