
---

```markdown
# STRIDE Summary

| Boundary                                      | Threat              | Impact                                            | Mitigation                                                          | Test hook                                            | Linked NFR/ADR        |
|-----------------------------------------------|---------------------|---------------------------------------------------|---------------------------------------------------------------------|------------------------------------------------------|-----------------------|
| FieldTechnician ↔ ConfigAdminAPI              | Spoofing            | Unauthorized edge configs                         | OIDC via AuthIdP, RBAC, TLS, audit logs                             | Attempt config with invalid/expired token            | NFR-010, ADR-005      |
| WildlifeScientist ↔ ConfigAdminAPI            | Tampering           | Wrong AOIs/thresholds                             | Versioned configs, signatures, two-person review (Assumed)          | Submit invalid config, expect reject/rollback        | NFR-010, ADR-005      |
| DroneOperator ↔ MissionPlannerDroneGateway    | Repudiation         | No trace of who approved missions                 | Signed approvals, mission IDs, immutable logs                       | Verify approvals recorded for sampled missions       | NFR-010, ADR-004      |
| EdgeCaptureScoring ↔ EdgeEventPolicy          | Tampering/EoP       | Injected or altered detection events              | Authenticated local channel, schema validation, bounds checks       | Fuzz events; confirm invalid events dropped          | NFR-003, ADR-001      |
| EdgeEventPolicy ↔ MissionPlannerDroneGateway  | Info Disclosure     | AOI leaks                                         | TLS, minimal AOI detail, encrypted disk at edge                     | Inspect payloads; ensure no unnecessary AOI details  | NFR-005*, ADR-004     |
| MissionPlannerDroneGateway ↔ DroneFleet       | Spoofing/Tampering  | Rogue missions, safety risk                       | Mutual auth, geofence enforcement, fail-safe rules                  | Push invalid mission; expect rejection               | NFR-008*, ADR-004     |
| EdgeEventPolicy ↔ WeatherAPI                  | Spoofing            | Unsafe missions on fake weather                   | HTTPS, provider pinning (Assumed), sanity checks                    | Feed bogus weather via test; verify gating           | ADR-004               |
| EdgeDataTransfer ↔ TapisOSC                   | Tampering           | Corrupt datasets, bad training/inference          | Chunk checksums, manifests, idempotent `batchId`, TLS               | Corrupt chunk in test; expect detect/retry/fail      | NFR-003, ADR-002      |
| EdgeDataTransfer ↔ TapisOSC                   | DoS                 | Backlog, data loss risk                           | Backoff, rate limiting, queue monitoring, alerting                  | Load test uploads; verify throttling and alerts      | NFR-002*, ADR-002     |
| CoreSystem ↔ NotificationSvc                  | Info Disclosure     | Leak sensitive AOIs in alerts                     | Redacted payloads, TLS, scoped API keys                             | Inspect alert samples for data minimization          | NFR-005*, ADR-005     |
| CoreSystem ↔ AuthIdP                          | Spoofing            | Unauthorized access                               | Standard OIDC/SAML, short token TTL, replay protection              | Replay token; expect rejection                       | ADR-005               |
| All Containers ↔ MonitoringTelemetry          | Tampering           | False metrics/logs hide failures                  | Append-only logs, restricted writers, integrity checks              | Attempt invalid writes; ensure blocked or flagged    | NFR-010, ADR-005      |
| Internal container-to-container (all)         | Elevation of Priv.  | One service controls others                       | Network segmentation, mTLS, least-privilege service accounts        | Pen-test lateral movement                            | NFR-011*, ADR-001     |

\*NFRs marked with * are assumed numeric targets to be finalized.
