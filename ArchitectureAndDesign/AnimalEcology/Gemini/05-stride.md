## STRIDE Summary

| **Boundary** | **Threat** | **Impact** | **Mitigation** | **Test Hook** | **Linked NFR/ADR** |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Actors -> Web Portal & API | **S**poofing | Attacker poses as Admin/Operator to approve/change missions. | RBAC, strong auth. | Test unauthorized access to admin/operator endpoints (FR-027). | NFR-SEC2, US-26 |
| Actors -> Web Portal & API | **T**ampering | Attacker changes AOI or approval status. | HTTPS (FR-022), config versioning (FR-028), audit logs (NFR-SEC2). | Fuzz API inputs; verify audit log on config change. | NFR-M1, US-29 |
| Actors -> Web Portal & API | **D**enial of Service | Attacker floods API, preventing mission approval or health reporting. | Rate limiting, WAF. | Load test key API endpoints (login, approval, health). | NFR-R2 |
| System (Planner) -> Parrot Drone | **T**ampering | Attacker injects malicious flight plan (e.g., fly-away, crash). | Mission plan checksums, strict geofence enforcement (FR-015), RTH failsafes (FR-015). | Test mission rejection with bad checksum; verify geofence RTH trigger. | NFR-R3, US-10 |
| System (Planner) -> Parrot Drone | **I**nformation Disclosure | Attacker sniffs drone telemetry or media feed. | Encrypted link (WPA2, SDK encryption). | Wi-Fi packet sniffing during active mission. | NFR-SEC1 |
| System (TransferSvc) -> OSC Storage | **T**ampering | Attacker corrupts media in transit. | HTTPS (FR-022), end-to-end checksum verification (FR-019). | MitM proxy to flip bits; verify transfer fails checksum. | NFR-P3, ADR-2 |
| System (TransferSvc) -> OSC Storage | **I**nformation Disclosure | Attacker steals sensitive wildlife (e.g., poacher) or human (US-34) data. | HTTPS (FR-022), encryption at rest on edge (FR-022, NFR-SEC1). | Verify TLS; verify edge disk encryption (if enabled). | NFR-SEC1, US-34 |
| System (Orchestrator) -> Tapis API | **S**poofing / **T**ampering | Attacker steals Tapis token to submit rogue jobs or delete data. | Store tokens in encrypted vault (FR-036); short-lived tokens; Tapis-side RBAC. | Verify token rotation (FR-036); test job submission with stolen/expired token. | NFR-SEC2, US-27 |
| System (Edge Agent) -> GPS/NTP | **T**ampering | Attacker spoofs GPS/NTP, causing incorrect timestamps or geofence failure. | Log large time/location jumps; cross-check GPS with known AOI bounds. | GPS/NTP spoofing test rig. | FR-001 |
06-sequences.md