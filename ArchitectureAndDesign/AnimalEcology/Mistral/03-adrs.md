
# Architecture Decision Records (ADRs)

## ADR 1: On-Device Scoring
- **Problem**: Limited compute and power on camera traps.
- **Options**:
  - **Option 1**: Use TensorFlow Lite for on-device scoring.
  - **Option 2**: Offload scoring to a cloud service.
- **Trade-offs**:
  - **Performance**: On-device scoring reduces latency but increases power usage.
  - **Reliability**: Cloud scoring is more reliable but requires network connectivity.
- **Decision**: Use TensorFlow Lite for on-device scoring.
- **Revisit Criteria**: If power constraints become unmanageable.

## ADR 2: Data Transfer Protocol
- **Problem**: Unreliable network connectivity.
- **Options**:
  - **Option 1**: Use SFTP with checksum verification.
  - **Option 2**: Use HTTP with retry logic.
- **Trade-offs**:
  - **Security**: SFTP is more secure but requires additional setup.
  - **Reliability**: HTTP with retries is simpler but less secure.
- **Decision**: Use SFTP with checksum verification.
- **Revisit Criteria**: If network reliability improves significantly.
