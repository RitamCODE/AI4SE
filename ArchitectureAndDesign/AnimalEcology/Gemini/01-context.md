C4Context
    title System Context Diagram - Wildlife Intrusion Response

    Person(fieldTech, "Field Technician", "Installs, maintains, and monitors edge hardware.")
    Person(droneOp, "Drone Operator", "Reviews, approves, and monitors drone missions for safety.")
    Person(scientist, "Wildlife Scientist", "Defines AOIs, sets detection thresholds, and reviews captured media.")
    Person(mlEng, "Data/ML Engineer", "Manages on-device models and downstream Tapis/OSC processing jobs.")
    Person(admin, "DevOps/Platform Admin", "Manages system configuration, user access (RBAC), and security.")

    System_Ext(drone, "Parrot Anafi Drone", "UAV that executes flight plans and captures media.")
    System_Ext(tapis, "Tapis API", "HPC orchestration platform used to submit and manage jobs.")
    System_Ext(osc, "OSC Storage", "Ohio Supercomputer Center secure storage for media and results.")
    System_Ext(time, "GPS/NTP Service", "Provides time synchronization for edge devices.")

    System_Boundary(boundary, "Wildlife Intrusion Response System") {
        System(sys, "Wildlife Intrusion Response System", "Detects intrusions, plans/executes drone follow-ups, and manages data/job pipeline to HPC.")
    }

    %% Actor Relationships
    Rel(fieldTech, sys, "Deploys & Monitors Health")
    Rel(droneOp, sys, "Approves/Aborts Missions")
    Rel(scientist, sys, "Configures AOIs/Thresholds & Reviews Media")
    Rel(mlEng, sys, "Manages Models & Job Templates")
    Rel(admin, sys, "Manages Configs & Secrets")

    %% System Relationships
    Rel(sys, drone, "Sends Mission Plans & Receives Media/Telemetry", "Parrot SDK/API")
    Rel(sys, osc, "Transfers Media Batches", "Resumable HTTPS/Tapis Files")
    Rel(sys, tapis, "Submits Orchestration Jobs", "REST API")
    Rel(sys, time, "Requests Time Sync", "NTP")