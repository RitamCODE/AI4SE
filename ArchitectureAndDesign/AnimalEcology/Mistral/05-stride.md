
# STRIDE Summary

| Boundary                | Threat                     | Impact                     | Mitigation                                      | Test Hook                     | Linked NFR/ADR       |
|-------------------------|----------------------------|----------------------------|-------------------------------------------------|-------------------------------|-----------------------|
| Camera Trap Controller  | Tampering                  | False positives/negatives | Secure boot, signed firmware                   | Penetration test              | NFR-001, ADR-1       |
| Data Transfer Agent     | Data corruption            | Loss of integrity          | Checksum verification, resumable transfers     | Fuzz testing                  | NFR-002, ADR-2       |
| Drone Manager           | Unauthorized flight        | Safety risk                | AOI geofencing, operator approval              | Flight boundary test          | NFR-003              |
