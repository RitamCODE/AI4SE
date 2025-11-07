C4Container
    title Container Diagram - Wildlife Intrusion Response

    %% Define People and External Systems
    Person(fieldTech, "Field Technician", "Deploys/maintains edge devices.")
    Person(droneOp, "Drone Operator", "Approves/monitors missions.")
    Person(scientist, "Wildlife Scientist", "Defines AOIs, reviews media.")
    Person(admin, "DevOps/Platform Admin", "Manages config and secrets.")
    System_Ext(drone, "Parrot Anafi Drone", "Executes missions, captures media.")
    System_Ext(tapis, "Tapis API", "Orchestrates HPC jobs.")
    System_Ext(osc, "OSC Storage", "Target for media files.")

    System_Boundary(edge_boundary, "Edge (Camera Trap / Base Station)") {
        Container(edgeAgent, "Edge Agent", "Python / RPi OS", "Captures, scores, filters, and buffers media/events. (FR-A, B, I)")
        Container(missionPlanner, "Mission Planner", "Python", "Generates AOI-constrained flight plans; manages drone safety checks. (FR-C, D)")
        Container(transferSvc, "Data Transfer Service", "Python / rsync-like", "Manages resumable, batched, checksum-verified uploads. (FR-E)")
    }

    System_Boundary(cloud_boundary, "Cloud (OSC / Cloud Platform)") {
        Container(webApi, "Web Portal & API", "React / FastAPI", "UI/API for config (AOIs, users, thresholds), mission approval, and status monitoring. (FR-G, H)")
        Container(orchestrator, "Pipeline Orchestrator", "Python / Tapis Client", "Submits Tapis jobs upon successful data transfer; tracks job status. (FR-F)")
        ContainerDb(db, "System Database", "PostgreSQL / JSON", "Stores AOIs, mission logs, job states, user roles, configs. (FR-G)")
    }

    %% Relationships
    %% User -> System
    Rel_L(fieldTech, edgeAgent, "Configures & Monitors Health", "SSH / Local UI")
    Rel(droneOp, webApi, "Approves Missions & Views Status", "HTTPS")
    Rel(scientist, webApi, "Manages AOIs & Reviews Events", "HTTPS")
    Rel(admin, webApi, "Manages RBAC & Secrets", "HTTPS")

    %% Edge Internal
    Rel(edgeAgent, missionPlanner, "Triggers Mission Plan", "Local IPC / REST")
    Rel(edgeAgent, transferSvc, "Queues Media Batch", "Local IPC / Filesystem")
    Rel(missionPlanner, transferSvc, "Queues Drone Media", "Local IPC / Filessystem")

    %% Edge <-> External
    Rel(missionPlanner, drone, "Sends Mission & Gets Telemetry", "Parrot SDK/API")

    %% Cloud Internal
    Rel(webApi, db, "Reads/Writes Config & State", "SQL")
    Rel(webApi, orchestrator, "Views Job Status", "API Call")
    Rel(orchestrator, db, "Updates Job Status", "SQL")

    %% Edge <-> Cloud
    Rel(edgeAgent, webApi, "Sends Health & Events", "HTTPS (async)")
    Rel(transferSvc, webApi, "Reports Transfer Status", "HTTPS")
    Rel(missionPlanner, webApi, "Requests Approval & Sends Logs", "HTTPS (async)")

    %% Cloud <-> External
    Rel(transferSvc, osc, "Uploads Media Batch", "Resumable HTTPS/Tapis Files")
    Rel(orchestrator, tapis, "Submits Job", "Tapis REST API")