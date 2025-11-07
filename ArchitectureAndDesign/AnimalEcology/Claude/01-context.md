## System Context - Wildlife Intrusion Response Pipeline
````mermaid
C4Context
    title System Context - Wildlife Intrusion Response Pipeline

    Person(fieldTech, "Field Technician", "Installs, maintains edge devices")
    Person(droneOp, "Drone Operator", "Approves/monitors missions")
    Person(scientist, "Wildlife Scientist", "Defines AOIs, reviews data")
    Person(mlEngineer, "ML Engineer", "Manages models, jobs")
    Person(platformAdmin, "Platform Admin", "Oversees security, ops")

    System(edgeSystem, "Edge-to-Cloud Wildlife System", "Detects intrusions, plans flights, transfers data, triggers ML jobs")

    System_Ext(tapis, "Tapis API", "Job orchestration platform")
    System_Ext(osc, "Ohio Supercomputer Center", "HPC for training/inference")
    System_Ext(weather, "Weather Service", "Wind, visibility data")
    System_Ext(ntp, "Time Service", "NTP/GPS sync")

    Rel(fieldTech, edgeSystem, "Configures devices, exports logs", "UI/CLI")
    Rel(droneOp, edgeSystem, "Approves missions, monitors flights", "UI/API")
    Rel(scientist, edgeSystem, "Defines AOIs, reviews events", "UI/API")
    Rel(mlEngineer, edgeSystem, "Deploys models, views job status", "API")
    Rel(platformAdmin, edgeSystem, "Manages RBAC, credentials", "Admin UI")

    Rel(edgeSystem, tapis, "Submits jobs with dataset refs", "HTTPS/REST")
    Rel(edgeSystem, osc, "Transfers media batches", "HTTPS/resumable")
    Rel(edgeSystem, weather, "Fetches forecast for safety checks", "HTTPS")
    Rel(edgeSystem, ntp, "Syncs timestamps", "NTP")

    Rel(tapis, osc, "Orchestrates compute jobs", "Internal")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
````

Caption:
External actors (technicians, operators, scientists, engineers, admins) interact with the Edge-to-Cloud system. The system coordinates with Tapis for job orchestration, OSC for data/compute, weather services for safety, and time services for sync.
Key Relationships:

Field technicians configure and maintain edge devices via UI/CLI
Drone operators approve and monitor missions in real-time
Wildlife scientists define survey areas and review captured data
ML engineers manage detection models and orchestrate compute jobs
Platform admins control access, credentials, and system observability
System transfers media to OSC and submits Tapis jobs automatically
Weather data gates flight safety decisions