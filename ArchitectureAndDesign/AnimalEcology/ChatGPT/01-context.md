# C4: System Context

```mermaid
C4Context
title Edge-to-Cloud Wildlife Intrusion Response — System Context

Person(FieldTechnician, "Field Technician", "Installs and maintains edge units.")
Person(DroneOperator, "Drone Operator", "Approves and supervises missions.")
Person(WildlifeScientist, "Wildlife Scientist", "Defines AOIs, species, thresholds.")
Person(DataMLEngineer, "Data/ML Engineer", "Manages models, jobs, datasets.")
Person(DevOpsAdmin, "DevOps/Platform Admin", "Operations, security, observability.")

System_Boundary(SYS, "EdgeToCloudWildlifeIntrusionResponseSystem") {
  System(CoreSystem, "Edge-to-Cloud Wildlife Intrusion Response System",
    "Camera-trap → Drone → OSC pipeline per SRS FR-001–FR-040.")
}

System_Ext(DroneFleet, "Drone Fleet Control/SDK", "Parrot Anafi or similar.")
System_Ext(TapisOSC, "Tapis/OSC Platform", "Jobs, storage, auth.")
System_Ext(WeatherAPI, "Weather / Regulatory API", "Weather and flight constraints.")
System_Ext(NotificationSvc, "Notification Service", "Email/SMS/Chat alerts.")
System_Ext(TimeSync, "Time Sync Service", "NTP/GPS time.")
System_Ext(AuthIdP, "Auth Identity Provider", "AuthN/RBAC.")
System_Ext(ResearchDataSink, "Research Data Consumers", "Downstream analytics tools.")

Rel(FieldTechnician, CoreSystem, "Configure, monitor, debug edge units", "HTTPS/Web UI/API")
Rel(DroneOperator, CoreSystem, "Review, approve, abort missions", "HTTPS/Web UI/API")
Rel(WildlifeScientist, CoreSystem, "Define AOIs, species, thresholds", "HTTPS/Web UI/API")
Rel(DataMLEngineer, CoreSystem, "Manage models, launch/inspect jobs", "HTTPS/Web UI/API")
Rel(DevOpsAdmin, CoreSystem, "Manage configs, keys, observability", "HTTPS/Web UI/API")

Rel(CoreSystem, DroneFleet, "Push mission plans, receive telemetry", "Drone SDK / HTTPS")
Rel(CoreSystem, TapisOSC, "Upload media, submit/manage jobs", "HTTPS/Tapis APIs")
Rel(CoreSystem, WeatherAPI, "Fetch weather and constraints", "HTTPS")
Rel(CoreSystem, NotificationSvc, "Send alerts & summaries", "HTTPS/Webhook")
Rel(CoreSystem, TimeSync, "Sync clocks", "NTP/GPS")
Rel(CoreSystem, AuthIdP, "Authenticate users", "OIDC/SAML")
Rel(CoreSystem, ResearchDataSink, "Expose curated datasets and metrics", "HTTPS/API")
