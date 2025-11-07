### Design: Sequence Diagrams

#### Flow 1: Event Detection to Mission Ready (Policy Auto-Launch)

This diagram shows the "happy path" for a high-priority detection that meets the pre-approved auto-launch criteria, critical for meeting NFR-P2.

```mermaid
sequenceDiagram
    title Flow 1: Event Detection to Mission Ready (Auto-Launch)
    participant EA as Edge Agent
    participant MP as Mission Planner
    participant API as Web Portal & API
    participant DB as System Database

    EA ->> EA: 1. Capture & Score (FR-002)
    EA ->> EA: 2. Filter & Coalesce (FR-006, US-36)
    EA ->> MP: 3. TriggerMissionPlan(Event, AOI)
    MP ->> API: 4. GetAOIDetails(AOI)
    API ->> DB: 5. read(AOI_polygon, flight_policy)
    DB -->> API: 6. AOIDetails
    API -->> MP: 7. AOIDetails
    MP ->> MP: 8. Generate Plan (FR-010)
    MP ->> API: 9. CheckAutoLaunchPolicy(Plan, AOI_policy)
    API ->> API: 10. Evaluate Rules (wind, daylight, etc) (US-09)
    API -->> MP: 11. {Policy: "Approve"}
    MP ->> API: 12. LogMission(Plan, "QueuedForLaunch")
    API ->> DB: 13. write(Mission)
    MP ->> MP: 14. Mission Ready (NFR-P2)


## Flow 2: Manual Mission Approval & Execution: 
This diagram shows the human-in-the-loop workflow where a mission requires manual operator approval (ADR-3).

```mermaid

sequenceDiagram
    title Flow 2: Manual Mission Approval & Execution
    actor DO as Drone Operator
    participant API as Web Portal & API
    participant MP as Mission Planner
    participant Drone as Parrot Anafi Drone
    participant DB as System Database

    %% This flow assumes Flow 1 happened but resulted in "Policy: 'ManualApproval'"
    DO ->> API: 1. GetPendingMissions()
    API ->> DB: 2. read(missions, status="PendingApproval")
    DB -->> API: 3. [Mission_123]
    API -->> DO: 4. ShowMission(Mission_123)
    DO ->> API: 5. ApproveMission(Mission_123) (US-08)
    API ->> DB: 6. update(Mission_123, status="Approved")
    API ->> MP: 7. NotifyMissionApproved(Mission_123)
    
    MP ->> MP: 8. Run Pre-Flight Checks (FR-014, US-11)
    MP ->> Drone: 9. Upload & StartMission(Plan)
    Drone -->> MP: 10. Telemetry(running)
    
    loop Media Capture (FR-016)
        Drone ->> MP: 11. MediaCaptured(geotag, timestamp)
    end
    
    Drone -->> MP: 12. MissionComplete(RTH)
    MP ->> API: 13. LogMissionComplete(Mission_123, media_list)
    API ->> DB: 14. update(Mission_123, status="Complete")

    alt Failsafe (US-12)
        Drone -->> MP: 15. FailsafeTriggered(LowBattery) (FR-015)
        MP ->> Drone: 16. CommandRTH()
        MP ->> API: 17. LogMissionFailed(Mission_123, "RTH")
    end

Flow 3: Data Transfer & Job Orchestration
This diagram shows the pipeline after drone media is collected, focusing on reliable transfer (ADR-2) and Tapis job submission.

```mermaid

sequenceDiagram
    title Flow 3: Data Transfer & Job Orchestration
    participant MP as Mission Planner
    participant DTS as Data Transfer Service
    participant OSC as OSC Storage
    participant PO as Pipeline Orchestrator
    participant Tapis as Tapis API
    participant DB as System Database

    MP ->> DTS: 1. QueueBatch(media_list, Mission_123)
    DTS ->> DTS: 2. Batch & Checksum (FR-019)
    
    loop Resumable Upload (FR-019, US-37)
        DTS ->> OSC: 3. UploadChunk(batch_id, chunk_N)
        opt Network Drop
            DTS ->> OSC: 4. UploadChunk(batch_id, chunk_N) [Retry]
        end
        OSC -->> DTS: 5. ChunkAck
    end
    
    DTS ->> OSC: 6. VerifyChecksum(batch_id)
    OSC -->> DTS: 7. Checksum OK
    
    DTS ->> PO: 8. NotifyBatchComplete(batch_id, location)
    PO ->> DB: 9. GetJobTemplate(Mission_123_AOI)
    DB -->> PO: 10. JobTemplate
    PO ->> Tapis: 11. SubmitJob(template, data_location) (FR-023, US-22)
    Tapis -->> PO: 12. Job_XYZ_Queued
    PO ->> DB: 13. update(Mission_123, job_status="Queued") (FR-024)
    
    loop Poll Job Status
        PO ->> Tapis: 14. GetJobStatus(Job_XYZ)
        Tapis -->> PO: 15. JobStatus(Running/Failed/Succeeded)
    end
    PO ->> DB: 16. update(Mission_123, job_status="Succeeded")

