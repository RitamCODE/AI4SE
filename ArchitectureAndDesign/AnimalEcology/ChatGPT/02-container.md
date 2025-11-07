
---

### 02-container.md

```markdown
# C4: Container

```mermaid
C4Container
title Edge-to-Cloud Wildlife Intrusion Response — Container View

System_Boundary(SYS, "EdgeToCloudWildlifeIntrusionResponseSystem") {

  Container(EdgeCaptureScoring, "EdgeCaptureScoring",
    "Edge service on camera traps (Python/C++).",
    "Capture images/video, run on-device models, maintain ring buffer. (FR-001–004, FR-018)")

  Container(EdgeEventPolicy, "EdgeEventPolicy",
    "Edge Event & Policy Engine (Go/Python).",
    "Filter detections, apply AOI/species thresholds, correlate events, trigger missions and transfers. (FR-006–013)")

  Container(MissionPlannerDroneGateway, "MissionPlannerDroneGateway",
    "Mission Planner & Drone Gateway (Python/SDK).",
    "Generate flight plans over AOIs, enforce safety rules, integrate with DroneFleet. (FR-014–017)")

  Container(EdgeDataTransfer, "EdgeDataTransfer",
    "Data Transfer Agent (Python).",
    "Chunked, checksum-verified, resumable uploads to OSC/Tapis; manage purge policies. (FR-018–021)")

  Container(TapisOSCOrchestrator, "TapisOSCOrchestrator",
    "Orchestration Service (Python/Java).",
    "Map datasets to training/inference jobs on Tapis/OSC; track status; retries. (FR-023–026)")

  Container(ConfigAdminAPI, "ConfigAdminAPI",
    "Config & Control API + Web UI (Node/Go).",
    "RBAC, AOIs, thresholds, model registry, mission rules, audit trail. (FR-027–029)")

  Container(MonitoringTelemetry, "MonitoringTelemetry",
    "Monitoring & Telemetry Service.",
    "Collect logs, metrics, health checks across all components. (FR-030–032)")
}

System_Ext(DroneFleet, "Drone Fleet Control/SDK", "")
System_Ext(TapisOSC, "Tapis/OSC Platform", "")
System_Ext(WeatherAPI, "Weather / Regulatory API", "")
System_Ext(NotificationSvc, "Notification Service", "")
System_Ext(AuthIdP, "Auth Identity Provider", "")

Rel(EdgeCaptureScoring, EdgeEventPolicy, "Detection events, scores", "IPC/gRPC")
Rel(EdgeEventPolicy, MissionPlannerDroneGateway, "Mission requests, AOI/Policy refs", "gRPC/HTTPS (signed)")
Rel(MissionPlannerDroneGateway, DroneFleet, "Upload mission, get status", "Drone SDK/HTTPS")
Rel(EdgeEventPolicy, EdgeDataTransfer, "Transfer tasks for selected media", "Local queue/gRPC")
Rel(EdgeDataTransfer, TapisOSC, "Chunked uploads, manifests", "HTTPS/Tapis Files API")
Rel(EdgeDataTransfer, TapisOSCOrchestrator, "Notify dataset ready", "HTTPS/Internal API")
Rel(TapisOSCOrchestrator, TapisOSC, "Create/manage jobs", "HTTPS/Tapis Jobs API")
Rel(ConfigAdminAPI, AuthIdP, "User authentication", "OIDC/SAML")
Rel(ConfigAdminAPI, EdgeEventPolicy, "Policies, AOIs, thresholds (versioned)", "HTTPS/Pull or PubSub")
Rel(ConfigAdminAPI, EdgeCaptureScoring, "Model configs", "HTTPS/Pull or PubSub")
Rel(ConfigAdminAPI, MissionPlannerDroneGateway, "Flight rules, geofences", "HTTPS/Pull or PubSub")
Rel(MonitoringTelemetry, EdgeCaptureScoring, "Logs, metrics", "Syslog/HTTPS")
Rel(MonitoringTelemetry, EdgeEventPolicy, "Logs, metrics", "Syslog/HTTPS")
Rel(MonitoringTelemetry, EdgeDataTransfer, "Logs, metrics", "Syslog/HTTPS")
Rel(MonitoringTelemetry, MissionPlannerDroneGateway, "Logs, metrics", "Syslog/HTTPS")
Rel(MonitoringTelemetry, TapisOSCOrchestrator, "Logs, metrics", "Syslog/HTTPS")
Rel(MonitoringTelemetry, ConfigAdminAPI, "Logs, metrics", "Syslog/HTTPS")
